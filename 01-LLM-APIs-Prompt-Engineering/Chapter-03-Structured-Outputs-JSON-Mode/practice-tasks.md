# Phase 1 → Chapter 3: Structured Outputs (JSON Mode) — Practice Tasks

---

## Task 1: Basic JSON Mode Try Karo (25 min)

1. Chapter 2 wale setup ko reuse karo (API key already ho gayi hogi).
2. Ye request banao:

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
        content: "Extract order items as JSON. Return JSON with 'items' array, each item has 'name' and 'quantity'."
      },
      { role: "user", content: "Mujhe teen samosa, do chai aur ek gulab jamun chahiye" }
    ],
    response_format: { type: "json_object" }
  })
});

const data = await response.json();
console.log(JSON.parse(data.choices[0].message.content));
```

3. Run karo, output verify karo — kya sahi quantities aur names extract hue?

**Deliverable:** Output paste karo, aur note karo agar koi item galat parse hua (jaise "gulab jamun" ko "gulab" aur "jamun" alag se treat kiya, ye common edge case hai).

---

## Task 2: Strict Schema Banao Aur Test Karo (30 min)

Task 1 wale code ko `json_schema` (strict mode) me convert karo — notes.md me diya schema example use karo.

Test karo edge cases ke saath:
1. Normal input: `"Do pizza aur ek coke"`
2. Ambiguous input: `"Kuch bhi bhej do accha sa"` (koi specific item nahi bola)
3. Multiple same items different phrasing: `"2 samosa, phir se ek samosa chahiye"` (kya model quantity 3 samjhega ya 2 alag entries banayega?)

**Deliverable:** Teeno cases ke outputs likho, observe karo model kaise handle karta hai ambiguous/edge cases ko.

---

## Task 3: Error Handling Add Karo (20 min)

Apne Task 1/2 wale code me proper error handling add karo:

```javascript
async function extractOrder(userMessage) {
  try {
    const response = await fetch(/* ... API call ... */);
    const data = await response.json();

    if (!data.choices || !data.choices[0]) {
      throw new Error("Invalid API response structure");
    }

    const parsed = JSON.parse(data.choices[0].message.content);

    // Validate structure manually bhi
    if (!parsed.items || !Array.isArray(parsed.items)) {
      throw new Error("Response missing 'items' array");
    }

    return parsed;
  } catch (error) {
    console.error("Order extraction failed:", error.message);
    return { items: [], error: true };
  }
}
```

Jaan-bujh kar ek malformed scenario simulate karo (jaise `max_tokens` bahut kam rakh do taaki JSON incomplete kat jaye) aur dekho tumhara error handling kaam karta hai ya nahi.

**Deliverable:** Working error-handled function, ek test case jaha error gracefully handle hua (crash nahi hua).

---

## Task 4: Apne QR System Me Apply Karo (25 min)

Apne QR Food Ordering System ke actual order-parsing logic ko dekho:

1. Kya wahan LLM se structured JSON already li ja rahi hai, ya text-parsing/regex use ho raha hai?
2. Agar structured output already use ho raha hai — schema kya hai, kya `strict` mode hai?
3. Agar nahi hai — socho (aur likho) ki isse structured output me convert karne se kya improvement hoga (reliability, error reduction)

**Deliverable:** Ek chhota technical note likho jo interview me directly use ho sake: "Mere QR System me order extraction is tarah handle hota hai..."