# Phase 1 → Chapter 5: Cost & Rate-Limit Awareness — Practice Tasks

---

## Task 1: Cost Calculator Banao (25 min)

Ek simple JavaScript function likho jo diye gaye pricing ke hisaab se cost calculate kare:

```javascript
function calculateCost(inputTokens, outputTokens, inputRatePerMillion, outputRatePerMillion) {
  const inputCost = (inputTokens / 1_000_000) * inputRatePerMillion;
  const outputCost = (outputTokens / 1_000_000) * outputRatePerMillion;
  return {
    inputCost: inputCost.toFixed(6),
    outputCost: outputCost.toFixed(6),
    totalCost: (inputCost + outputCost).toFixed(6)
  };
}

// Test karo:
console.log(calculateCost(180, 50, 3, 15));
```

Ab in scenarios ke liye calculate karo:
1. 100 requests/day, average 180 input + 50 output tokens — monthly cost kya hoga?
2. 10,000 requests/day (scaled up scenario) — monthly cost kya hoga?

**Deliverable:** Dono scenarios ke calculations, aur ek observation — scale hone pe cost kaise grow karta hai (linear hai ya nahi).

---

## Task 2: Real API Se `usage` Field Nikal Kar Log Karo (20 min)

Chapter 2 wala apna API call code lo, usme ye function add karo:

```javascript
function logUsage(usage) {
  const cost = calculateCost(usage.input_tokens || usage.prompt_tokens, 
                               usage.output_tokens || usage.completion_tokens, 
                               3, 15); // apne actual rates daalo
  console.log(`Request cost: $${cost.totalCost} | Input: ${usage.input_tokens} tokens | Output: ${usage.output_tokens} tokens`);
}
```

3-4 different requests bhejo (alag-alag length ke messages ke saath), har baar cost log karo.

**Deliverable:** Ek chhota table — Request | Input Tokens | Output Tokens | Cost — jisme dikhe ki lambe messages/response se cost kaise badhta hai.

---

## Task 3: Exponential Backoff Implement Karo (30 min)

Notes.md wala `callLLMWithRetry` function apne code me implement karo.

Test karo (simulate karne ke liye):
1. Ek fake function banao jo pehle 2 calls pe `429` error throw kare, teesri call pe success de:

```javascript
let attemptCount = 0;
async function fakeAPICall() {
  attemptCount++;
  if (attemptCount < 3) {
    return { status: 429 };
  }
  return { status: 200, ok: true, json: async () => ({ result: "success" }) };
}
```

2. Apne retry logic ko is fake function ke saath test karo, console.log se dekho ki wait times kaise badh rahe hain (1s, 2s, 4s).

**Deliverable:** Working retry logic, console output jisme retry attempts aur wait times dikhein.

---

## Task 4: Apne Project Me Cost/Rate-Limit Audit Karo (20 min)

Apne QR System (ya jo bhi project LLM use karta hai) me check karo:

1. Kya kahi retry logic hai agar API call fail ho jaye? Agar nahi, ye ek gap hai — note kar lo.
2. Kya `max_tokens` kahi unusually high set hai bina reason ke?
3. Kya usage/cost logging ho rahi hai kahi? Agar nahi, socho kaha add karoge.
4. Estimate karo — agar tumhara app real users ke saath 1000 requests/day handle kare, monthly cost approximately kitni hogi (Task 1 wala calculator use karke)?

**Deliverable:** Ek gap-analysis note — "Mere project me ye missing hai, production-ready banane ke liye ye add karna chahiye." Ye interview me directly bol sakte ho jab poochein "tumhare project me kya improve kar sakte ho."

---

**Phase 1 Complete!** Ab tumne LLM APIs, prompt engineering, aur production concerns (cost/rate-limits) cover kar liye hain. Agla phase — Function/Tool Calling — jaha tum LLM ko actual actions perform karna sikhaoge.