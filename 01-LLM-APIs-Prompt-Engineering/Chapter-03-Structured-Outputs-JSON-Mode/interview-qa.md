# Phase 1 → Chapter 3: Structured Outputs (JSON Mode) — Interview Q&A

---

**Q1. Structured Output/JSON mode kya hota hai aur kyu zaroori hai?**

Ye ek technique hai jisse LLM ko force karte hain ki wo free-flowing text ki jagah valid, predictable JSON return kare — ek defined schema follow karte hue. Zaroori hai kyunki production apps me LLM output ko directly database/downstream code me use karna hota hai, aur fragile text-parsing/regex se reliable nahi hota.

---

**Q2. `response_format: { type: "json_object" }` aur `json_schema` mode me kya difference hai?**

`json_object` sirf ye guarantee deta hai ki output valid JSON hoga, lekin fields kya honge ye guarantee nahi. `json_schema` (strict mode) ek exact schema enforce karta hai — model guaranteed usi structure ko follow karega, extra ya missing fields nahi aayenge. Production ke liye `json_schema` zyada reliable hai.

---

**Q3. Anthropic me structured output kaise milta hai, jabki direct JSON schema mode nahi hai?**

Tool calling ka use karke — ek "fake tool" define karte hain jiska `input_schema` tumhara desired output structure hota hai, aur `tool_choice` se us tool ko force karte hain. Model tool call ke through structured data return karta hai jo `content` array me `tool_use` block ke andar milta hai.

---

**Q4. Kab structured output use nahi karna chahiye?**

General conversational replies ya long-form creative content (blog, story) ke liye — structured format creativity aur natural flow ko restrict karta hai, user experience kharab ho sakta hai. Structured output un cases ke liye hai jaha data ko programmatically process karna hai (order extraction, form parsing, classification).

---

**Q5. Production code me `JSON.parse()` use karte waqt kya precaution leni chahiye?**

Hamesha try-catch me wrap karna chahiye. Kyunki (khaaskar bina strict schema ke) model rarely malformed JSON de sakta hai, aur agar parse fail ho jaye bina error handling ke, poora request crash ho sakta hai.

---

**Q6. `additionalProperties: false` schema me set karna kyu important hai?**

Ye ensure karta hai model sirf defined fields hi return kare, koi extra unexpected field add na kare. Bina iske, model kabhi kabhi additional fields daal sakta hai jo downstream code (jo specific fields expect karta hai) ko break kar sakte hain.

---

**Q7. Apne QR System ke context me — order extraction ke liye structured output kaise use karoge, real example do.**

User natural language me order bolta hai ("do paneer tikka aur ek lassi"), system prompt+schema define karta hai `items` array with `name` and `quantity`. LLM structured JSON return karta hai jo directly parse karke order object database me insert ho sakta hai, bina manual text-parsing/regex likhe.

---

**Q8. Structured output aur tool calling me kya conceptual similarity hai?**

Dono me ek defined schema hota hai jise LLM follow karta hai. Tool calling me ye schema "function ke parameters" define karta hai jo LLM ko call karna hai; structured output me ye seedha "response format" define karta hai. Anthropic dono cases me isi tool-use mechanism ko reuse karta hai.

---

**Q9. Agar `json_object` mode use kiya lekin prompt me "JSON" word mention nahi kiya, toh kya hoga?**

OpenAI jaisi APIs is case me error de sakti hain ya unexpected behavior aa sakta hai — providers explicitly require karte hain ki prompt (system ya user message) me JSON output ka mention ho jab JSON mode enable ho.

---

**Q10. Classification tasks (jaise spam detection) ke liye structured output kyu helpful hai?**

Kyunki consistent, machine-readable labels milte hain (jaise `{"label": "spam", "confidence": 0.92}`), jinhe directly conditional logic me use kar sakte hain — bina free-text response se manually keyword-match karke label nikalne ki zarurat.