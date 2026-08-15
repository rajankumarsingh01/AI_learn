# Phase 1 → Chapter 2: OpenAI & Anthropic API

## Kya/Kyu/Kaise

**Kya hai ye?**
OpenAI aur Anthropic dono companies apne LLMs (GPT models, Claude models) ko ek **REST API** ke through expose karti hain. Matlab tum HTTP request bhejo (jaisa tum kisi normal backend API ko bhejte ho), aur response me LLM ka generated text wapas milta hai.

**Kyu seekhna zaroori hai?**
Tumhare QR System me already isi tarah ka API call ho raha hoga (kisi LLM provider ka). Lekin abhi tak tumne "structure" nahi samjha — messages array kya hota hai, system prompt kaha jata hai, roles kya matter karte hain. Ye samjhe bina LangChain jaisa framework use karoge toh sirf "copy-paste" karoge, samjhoge nahi ki andar kya ho raha hai.

**Kaise kaam karta hai?**
Tum ek JSON payload bhejte ho jisme messages ka array hota hai (har message ka ek "role" hota hai — system/user/assistant), API provider ke server pe wo jata hai, LLM process karta hai, aur JSON response wapas aata hai jisme generated text hota hai.

---

## 1. Basic Request Structure

Dono providers (OpenAI aur Anthropic) ka structure similar hai, thoda naming difference hai.

### OpenAI API example (Node.js):
```javascript
const response = await fetch("https://api.openai.com/v1/chat/completions", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": `Bearer ${process.env.OPENAI_API_KEY}`
  },
  body: JSON.stringify({
    model: "gpt-4o",
    messages: [
      { role: "system", content: "You are a helpful assistant for a food ordering app." },
      { role: "user", content: "What vegetarian options do you have?" }
    ],
    temperature: 0.3,
    max_tokens: 300
  })
});

const data = await response.json();
console.log(data.choices[0].message.content);
```

### Anthropic API example (Node.js):
```javascript
const response = await fetch("https://api.anthropic.com/v1/messages", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "x-api-key": process.env.ANTHROPIC_API_KEY,
    "anthropic-version": "2023-06-01"
  },
  body: JSON.stringify({
    model: "claude-sonnet-4-6",
    max_tokens: 300,
    system: "You are a helpful assistant for a food ordering app.",
    messages: [
      { role: "user", content: "What vegetarian options do you have?" }
    ]
  })
});

const data = await response.json();
console.log(data.content[0].text);
```

**Key difference:** OpenAI me system prompt bhi `messages` array ke andar ek role hota hai. Anthropic me system prompt **alag se ek top-level field** hota hai, `messages` array ke bahar.

---

## 2. Roles — Ye kya matter karte hain

Har message ka ek role hota hai, aur LLM in roles ko differently treat karta hai:

| Role | Kaam |
|---|---|
| **system** | Model ka "behavior instructions" — ye kaisa assistant hai, kya karna hai, kya nahi karna. Ek baar set hota hai, poori conversation ke liye apply hota hai. |
| **user** | Actual insaan ka message — jo tumhara app user type kar raha hai. |
| **assistant** | LLM ka pichla response — jab tum multi-turn conversation bana rahe ho, toh purane LLM responses ko wapas bhejna padta hai taaki model ko "yaad" rahe usne kya kaha tha. |

### Example — Multi-turn conversation:
```javascript
messages: [
  { role: "system", content: "You are a customer support bot for Kaksha coaching platform." },
  { role: "user", content: "Mera payment fail ho gaya" },
  { role: "assistant", content: "Samajh gaya. Kya aap bata sakte hain kaunsa payment method use kiya tha?" },
  { role: "user", content: "UPI se try kiya tha" }
]
```

Notice karo — pichla assistant response bhi bheja gaya hai. **Ye zaroori hai** kyunki LLM ke paas khud ki memory nahi hoti (Chapter 1 yaad karo — context window). Har request ek fresh call hai, isliye poori history explicitly bhejni padti hai.

---

## 3. Important Parameters

| Parameter | Kya karta hai | Typical value |
|---|---|---|
| `model` | Kaunsa specific model use karna hai (gpt-4o, claude-sonnet-4-6, etc.) | Task complexity ke hisaab se |
| `temperature` | Randomness control (Chapter 1 me detail padha) | 0-0.3 factual, 0.7+ creative |
| `max_tokens` | Response me max kitne tokens generate honge | Task ke hisaab se, cost bhi control karta hai |
| `system` (Anthropic) / system role (OpenAI) | Behavior instructions | Har use-case specific |

---

## 4. Response Structure Samajhna

### OpenAI response:
```json
{
  "choices": [
    {
      "message": {
        "role": "assistant",
        "content": "Humare paas paneer tikka, dal makhani..."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 45,
    "completion_tokens": 60,
    "total_tokens": 105
  }
}
```

### Anthropic response:
```json
{
  "content": [
    { "type": "text", "text": "Humare paas paneer tikka, dal makhani..." }
  ],
  "stop_reason": "end_turn",
  "usage": {
    "input_tokens": 45,
    "output_tokens": 60
  }
}
```

**Dhyan do:** `usage` field har response me aata hai — yehi batata hai kitne tokens use hue, jisse tum cost calculate kar sakte ho (Chapter 5 me detail me).

---

## 5. `finish_reason` / `stop_reason` — Ye kyu check karna zaroori hai

Ye field batata hai response **kyu** stop hua:

- `stop` / `end_turn` — Model ne khud decide kiya response complete hai. Normal, healthy case.
- `length` / `max_tokens` — `max_tokens` limit hit ho gayi, response beech me hi kat gaya. **Ye bug ka sign hai** — production me isse handle karna padta hai (max_tokens badhao ya user ko warn karo response incomplete hai).
- `content_filter` — Safety filter trigger hua.

Production code me hamesha ye check karna chahiye, warna incomplete response silently user ko chala jayega.

---

## Common Mistake (Interview me pooche jate hain)

1. **System prompt ko har message ke saath repeat na bhejna** — kuch log system prompt sirf first message me bhejte hain, socchte hain LLM "yaad" rakhega. Galat — bina explicit context ke, agle request me model ko system prompt phir se bhejna padta hai (jab tak koi caching mechanism na ho).
2. **`max_tokens` ko bahut kam rakhna** — response beech me kat jata hai, `finish_reason: "length"` aata hai, aur developer confuse hota hai ki "LLM adhoora jawab de raha hai."
3. **API key ko frontend code me expose karna** — Hamesha backend se hi API call karo, kabhi frontend/client-side se directly LLM API mat call karo, warna API key leak ho jayegi.