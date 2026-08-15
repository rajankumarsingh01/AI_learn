# Phase 1 → Chapter 2: OpenAI & Anthropic API — Practice Tasks

---

## Task 1: Real API Call — Node.js se (30 min)

1. Ek naya folder banao: `chapter-02-practice`
2. `npm init -y` chalao, `.env` file banao
3. OpenAI ya Anthropic (jo bhi API key available ho) ka free-tier/trial key le lo (agar nahi hai, structure samajhne ke liye code likh lo, actual call optional hai)
4. Ye script likho:

```javascript
require('dotenv').config();

async function askLLM(userMessage) {
  const response = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "x-api-key": process.env.ANTHROPIC_API_KEY,
      "anthropic-version": "2023-06-01"
    },
    body: JSON.stringify({
      model: "claude-sonnet-4-6",
      max_tokens: 200,
      system: "You are a helpful assistant for Kaksha, a coaching institute platform. Answer in Hinglish.",
      messages: [
        { role: "user", content: userMessage }
      ]
    })
  });

  const data = await response.json();
  console.log("Response:", data.content[0].text);
  console.log("Tokens used:", data.usage);
}

askLLM("Mujhe apna installment payment schedule check karna hai");
```

5. Run karo aur output dekho — `usage` field me tokens count note karo.

**Deliverable:** Console output ka screenshot ya copy, saath me apna observation ki input tokens vs output tokens kitne aaye.

---

## Task 2: Multi-turn Conversation Banao (25 min)

Upar wale script ko extend karo taaki ye 3-message conversation handle kare:

```javascript
const conversationHistory = [
  { role: "user", content: "Mera installment payment fail ho gaya tha" },
  { role: "assistant", content: "Samajh gaya, kaunsa payment method use kiya tha?" },
  { role: "user", content: "UPI se try kiya tha" }
];
```

Isse API ko bhejo aur dekho response kaisa aata hai — kya wo pichle context ko samajh ke relevant jawab deta hai?

**Deliverable:** Verify karo ki agar tum pichla `assistant` message hata do array se, kya response quality/relevance change hoti hai. Dono outputs compare karo aur likho difference.

---

## Task 3: `finish_reason` Trigger Karo Jaan-bujh Kar (15 min)

1. `max_tokens: 10` set karo (bahut kam)
2. Ek aisa prompt do jiska lamba answer chahiye ho: `"Explain how RAG systems work in detail"`
3. Response me `finish_reason` / `stop_reason` field check karo — `"length"` aana chahiye
4. Ab `max_tokens: 500` kar do, dobara run karo — is baar `"stop"`/`"end_turn"` aana chahiye

**Deliverable:** Dono responses compare karo, likho kya difference dikha (incomplete sentence vs complete response).

---

## Task 4: Apne QR System Code Audit Karo (20 min)

Apne QR Food Ordering System ke code me jao jaha LLM API call hoti hai:

1. Kaunsa provider use ho raha hai (OpenAI/Anthropic/koi aur)?
2. System prompt kaha define hai aur kya likha hai usme?
3. Conversation history kaise maintain ho rahi hai — Redis se fetch karke `messages` array me daali ja rahi hai?
4. `max_tokens` kya set hai, aur kya kabhi `finish_reason: "length"` wala issue face kiya tha?

**Deliverable:** Ek chhota summary likho apne code ke actual implementation ka — ye directly interview answer ban jayega jab poochenge "apna LLM integration explain karo."