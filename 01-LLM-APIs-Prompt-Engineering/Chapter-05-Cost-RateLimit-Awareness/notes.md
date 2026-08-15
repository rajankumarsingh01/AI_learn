# Phase 1 → Chapter 5: Cost & Rate-Limit Awareness

## Kya/Kyu/Kaise

**Kya hai ye?**
LLM APIs free nahi hain — har request ke tokens (input + output) ke hisaab se charge hota hai. Aur har provider ka **rate limit** bhi hota hai — matlab ek fixed time window me kitni requests/tokens allow hain. Ye chapter samjhata hai cost kaise calculate hoti hai, aur rate limits ko production code me kaise handle karte hain.

**Kyu zaroori hai?**
Ye woh part hai jo zyadatar fresher developers **ignore kar dete hain** — wo bas "API call kar do, kaam ho gaya" soch lete hain. Lekin production me agar cost/rate-limit awareness nahi hai, toh (a) tumhara AWS/API bill acha-khasa aa sakta hai without warning, (b) app crash ho sakta hai jab traffic spike ho aur rate limit hit ho jaye. Interview me ye pooche jane wale sabse "practical/senior" level sawalo me se ek hai — freshers isse avoid karte hain, tum nahi karoge.

**Kaise kaam karta hai?**
Providers `usage` field response me batate hain kitne tokens use hue (Chapter 2 me dekha tha). Rate limits HTTP headers me communicate hote hain, aur jab exceed ho jate hain toh `429 Too Many Requests` error milta hai — jisse gracefully handle karna padta hai (retry logic, exponential backoff).

---

## 1. Cost Calculation — Real Numbers Se Samjho

Providers **per-million-tokens** basis pe charge karte hain, input aur output alag rates pe (output usually zyada expensive hota hai kyunki generation compute-heavy hai).

### Example pricing structure (illustrative, actual rates provider/model ke hisaab se check karo):
```
Model: Mid-tier model
Input:  $3 per million tokens
Output: $15 per million tokens
```

### Real calculation:
```
Ek QR System order-extraction request:
- System prompt: ~150 tokens
- User message: ~30 tokens
- Total input: ~180 tokens
- Output (JSON response): ~50 tokens

Cost per request:
Input cost  = (180 / 1,000,000) × $3  = $0.00054
Output cost = (50 / 1,000,000) × $15  = $0.00075
Total per request ≈ $0.00129 (~₹0.11)

Agar din me 500 orders aaye:
500 × $0.00129 = $0.645/day ≈ ₹54/day ≈ ₹1,620/month
```

**Interview-relevant insight:** Ye numbers chhote lagte hain, lekin scale hone pe (jaise 50,000 requests/day) ye significant ho jate hain. Isliye cost-optimization ek real engineering concern hai, "nice-to-have" nahi.

---

## 2. Cost Optimization Techniques

| Technique | Kaise kaam karta hai |
|---|---|
| **Shorter system prompts** | Har request ke saath system prompt jata hai — usse concise rakhna repeated cost bachata hai |
| **Smaller/cheaper models for simple tasks** | Sabhi task ke liye sabse powerful (expensive) model use na karo — classification jaisa simple task chhote model se ho sakta hai |
| **`max_tokens` sensibly set karna** | Bahut zyada `max_tokens` set karne se accidental long outputs generate ho sakte hain jo cost badhate hain |
| **Caching** | Agar same/similar query repeat ho rahi hai, response cache karo — LLM API dobara mat call karo |
| **Batching (jaha applicable)** | Multiple independent requests ko batch API (agar provider support kare) se cheaper rate pe process karna |

---

## 3. Rate Limits Samajhna

Rate limits do tarah ke hote hain:
- **RPM (Requests Per Minute)** — ek minute me kitni API calls kar sakte ho
- **TPM (Tokens Per Minute)** — ek minute me kitne total tokens process kar sakte ho

### Response headers (typical example):
```
x-ratelimit-limit-requests: 500
x-ratelimit-remaining-requests: 487
x-ratelimit-limit-tokens: 200000
x-ratelimit-remaining-tokens: 195000
x-ratelimit-reset-requests: 12s
```

Production code me ye headers check karke tum proactively slow down kar sakte ho, bina error aane ka wait kiye.

---

## 4. Rate Limit Errors Handle Karna — Exponential Backoff

Jab rate limit exceed hota hai, API `429 Too Many Requests` error deti hai. Best practice hai **exponential backoff with retry**:

```javascript
async function callLLMWithRetry(payload, maxRetries = 3) {
  let attempt = 0;

  while (attempt < maxRetries) {
    try {
      const response = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "x-api-key": process.env.ANTHROPIC_API_KEY,
          "anthropic-version": "2023-06-01"
        },
        body: JSON.stringify(payload)
      });

      if (response.status === 429) {
        const waitTime = Math.pow(2, attempt) * 1000; // 1s, 2s, 4s...
        console.log(`Rate limited. Retrying after ${waitTime}ms`);
        await new Promise(resolve => setTimeout(resolve, waitTime));
        attempt++;
        continue;
      }

      if (!response.ok) {
        throw new Error(`API error: ${response.status}`);
      }

      return await response.json();

    } catch (error) {
      attempt++;
      if (attempt >= maxRetries) throw error;
    }
  }

  throw new Error("Max retries exceeded for LLM API call");
}
```

**Kyu "exponential" backoff:** Har retry pe wait time double karte hain (1s → 2s → 4s) — isse agar server already overloaded hai, tum usse aur zyada requests bhejke situation worse nahi karte. Ye standard distributed-systems pattern hai (sirf AI APIs tak limited nahi).

---

## 5. Production Monitoring — Log Karna Zaroori Hai

Har LLM call ka `usage` data log karna chahiye taaki:
- Daily/monthly cost track ho sake
- Anomalies detect ho sake (achanak zyada usage — bug ya abuse ka sign ho sakta hai)
- Budget alerts set kar sako

```javascript
function logLLMUsage(requestId, usage) {
  console.log({
    requestId,
    inputTokens: usage.input_tokens,
    outputTokens: usage.output_tokens,
    estimatedCost: calculateCost(usage),
    timestamp: new Date().toISOString()
  });
  // Production me: database/monitoring service (Grafana, Datadog, etc.) me bhejo
}
```

**Tumhare Monitoring System project se direct connection:** Tumne already ek Observability platform banaya hai — wahi concept yaha LLM-specific metrics ke liye apply hota hai. Interview me ye connect kar sakte ho: "Maine apne Observability platform me jo monitoring patterns seekhe, wahi LLM cost-tracking me bhi apply kiye."

---

## Common Mistakes (Interview me pooche jate hain)

1. **Rate limit error ko silently fail hone dena** — bina retry logic ke, ek spike traffic pe pura feature "down" dikhega users ko, jabki simple retry se recover ho sakta tha.
2. **Cost tracking na karna production me** — bina monitoring ke, ek buggy loop (jo accidentally LLM ko baar-baar call kare) unnoticed reh sakta hai jab tak bill nahi aata.
3. **Har task ke liye sabse expensive/powerful model use karna** — simple classification/extraction tasks ke liye bhi flagship model use karna unnecessary cost hai, jab chhota model kaafi hota.
4. **`max_tokens` ko bahut high default set karna "safe side" soch kar** — isse worst-case cost bhi high ho jata hai, especially agar model kabhi verbose/repetitive output de de.