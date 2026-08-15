# Phase 1 → Chapter 3: Structured Outputs (JSON Mode)

## Kya/Kyu/Kaise

**Kya hai ye?**
Normally LLM free-flowing text generate karta hai — jaisa insaan likhta hai. Lekin production apps me tumhe **predictable, parseable data** chahiye hoti hai — jaise ek order object, ek form response, ek API-ready JSON. Structured Outputs (ya JSON Mode) ek technique hai jisse tum LLM ko force karte ho ki wo **hamesha valid JSON** return kare, ek fixed schema follow karte hue.

**Kyu zaroori hai?**
Socho tumhare QR System ka chatbot user se poochta hai "kya order karna hai," aur user bolta hai "do paneer tikka aur ek lassi." Agar LLM plain text me jawab de ("Aapne 2 paneer tikka aur 1 lassi order kiya hai") toh tumhara backend code isse parse nahi kar sakta reliably — tumhe ye chahiye:

```json
{
  "items": [
    { "name": "paneer tikka", "quantity": 2 },
    { "name": "lassi", "quantity": 1 }
  ]
}
```

Isi structured format ko seedha database me daal sakte ho, order create kar sakte ho, bina fragile text-parsing/regex ke.

**Kaise kaam karta hai?**
API request me tum bolte ho "response JSON format me chahiye," aur ideally ek **schema** bhi define karte ho (kaunse fields chahiye, kaunse type ke). Model us schema ko follow karne ki koshish karta hai. Modern APIs (OpenAI, Anthropic) me ye guarantee bhi milti hai ki output **valid JSON hi hoga**, malformed nahi.

---

## 1. Basic JSON Mode — OpenAI

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
      {
        role: "system",
        content: "Extract order items from user message. Return JSON with 'items' array, each item has 'name' and 'quantity'."
      },
      { role: "user", content: "Mujhe do paneer tikka aur ek lassi chahiye" }
    ],
    response_format: { type: "json_object" }  // <- Ye JSON mode force karta hai
  })
});

const data = await response.json();
const parsedOrder = JSON.parse(data.choices[0].message.content);
console.log(parsedOrder);
// { items: [ { name: "paneer tikka", quantity: 2 }, { name: "lassi", quantity: 1 } ] }
```

**Important:** `response_format: { type: "json_object" }` set karne ke baad, system/user prompt me explicitly "JSON" word mention karna zaroori hai — warna API error de sakti hai.

---

## 2. Schema-Enforced Output (Structured Outputs) — Zyada Reliable

Sirf "JSON do" bolna kaafi nahi hota — kabhi kabhi model extra/missing fields daal deta hai. **Structured Outputs** (OpenAI ka newer feature) ek exact schema enforce karta hai using JSON Schema:

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
      { role: "system", content: "Extract order items from user message." },
      { role: "user", content: "Mujhe do paneer tikka aur ek lassi chahiye" }
    ],
    response_format: {
      type: "json_schema",
      json_schema: {
        name: "order_extraction",
        strict: true,
        schema: {
          type: "object",
          properties: {
            items: {
              type: "array",
              items: {
                type: "object",
                properties: {
                  name: { type: "string" },
                  quantity: { type: "integer" }
                },
                required: ["name", "quantity"],
                additionalProperties: false
              }
            }
          },
          required: ["items"],
          additionalProperties: false
        }
      }
    }
  })
});
```

Is approach me `strict: true` ka matlab hai — model **guaranteed** isi schema ko follow karega, extra field ya missing field nahi aayega. Ye production ke liye best practice hai.

---

## 3. Anthropic me Structured Output — Tool Use Trick

Anthropic ke paas directly "json_schema" mode nahi hai (is roadmap ke time tak), lekin **tool calling** ka use karke same effect milta hai — ek "fake tool" define karo jiska schema tumhara desired output structure ho:

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
    tools: [
      {
        name: "extract_order",
        description: "Extract order items from user message",
        input_schema: {
          type: "object",
          properties: {
            items: {
              type: "array",
              items: {
                type: "object",
                properties: {
                  name: { type: "string" },
                  quantity: { type: "integer" }
                },
                required: ["name", "quantity"]
              }
            }
          },
          required: ["items"]
        }
      }
    ],
    tool_choice: { type: "tool", name: "extract_order" },  // force karta hai isi tool ko use kare
    messages: [
      { role: "user", content: "Mujhe do paneer tikka aur ek lassi chahiye" }
    ]
  })
});

const data = await response.json();
const toolUseBlock = data.content.find(block => block.type === "tool_use");
console.log(toolUseBlock.input);
// { items: [ { name: "paneer tikka", quantity: 2 }, { name: "lassi", quantity: 1 } ] }
```

**Ye pattern Chapter 4 (Function/Tool Calling) me deeply padhoge** — abhi bas samajh lo ki structured output aur tool calling technically related concepts hain.

---

## 4. Kab Structured Output Use Karna Chahiye

| Scenario | Structured Output? |
|---|---|
| Order extraction, form data parsing | Haan — zaroori |
| Database me directly insert karna hai LLM output | Haan — zaroori |
| General chatbot conversation reply | Nahi — natural text better UX deta hai |
| Classification (spam/not-spam, sentiment) | Haan — helpful, consistent labels milte hain |
| Long-form content generation (blog, story) | Nahi — structure creativity ko restrict karega |

---

## Common Mistakes (Interview me pooche jate hain)

1. **`JSON.parse()` bina try-catch ke use karna** — kabhi kabhi (rare but possible without strict schema) model malformed JSON de sakta hai. Production code me hamesha try-catch wrap karo.
2. **Schema me `additionalProperties: false` na set karna** — isse model kabhi kabhi extra unexpected fields add kar deta hai jo tumhare downstream code ko break kar sakte hain.
3. **JSON mode use karke bhi prompt me "JSON" word na likhna** — OpenAI jaise providers require karte hain ki prompt me explicitly JSON output ka mention ho, warna request fail ho sakti hai.