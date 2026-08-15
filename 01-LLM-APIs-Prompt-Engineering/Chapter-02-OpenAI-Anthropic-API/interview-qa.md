# Phase 1 → Chapter 2: OpenAI & Anthropic API — Interview Q&A

---

**Q1. LLM API call ka basic structure kya hota hai?**

Ek POST request jisme JSON body hota hai — usme `model` (kaunsa model use karna hai), `messages` array (conversation history, role ke saath), aur parameters jaise `temperature`, `max_tokens`. Authorization header me API key jati hai. Response me generated text `choices`/`content` field ke andar aata hai.

---

**Q2. System, user, aur assistant role me kya difference hai?**

`system` role model ka behavior define karta hai (instructions) — poori conversation ke liye set hota hai. `user` role actual insaan ka message hota hai. `assistant` role LLM ke pichle responses store karta hai, jo multi-turn conversation me wapas bhejne padte hain taaki model ko conversation history "yaad" rahe — kyunki LLM khud koi memory maintain nahi karta.

---

**Q3. Multi-turn conversation me purane assistant responses wapas kyu bhejne padte hain?**

Kyunki LLM stateless hai — har API call independent hoti hai, model ke paas koi built-in memory nahi hoti (Chapter 1: context window). Agar purane messages nahi bheje, toh model ko pata hi nahi chalega conversation me pehle kya baat hui thi.

---

**Q4. OpenAI aur Anthropic ke API structure me main difference kya hai?**

OpenAI me system prompt `messages` array ke andar ek role ke roop me jata hai. Anthropic me system prompt ek alag top-level `system` field hota hai, `messages` array se bahar. Dono me `messages` array conversation history carry karta hai.

---

**Q5. `finish_reason: "length"` aaye toh iska matlab kya hai aur kaise handle karoge?**

Iska matlab hai response `max_tokens` limit ke wajah se beech me kat gaya, model ne khud response complete nahi kiya. Production me isse detect karke ya toh `max_tokens` badhana chahiye, ya user ko batana chahiye response incomplete hai, ya continuation request bhejni chahiye.

---

**Q6. API key ko frontend/client-side code me kyu nahi rakhna chahiye?**

Kyunki frontend code browser me publicly visible/inspectable hota hai — koi bhi API key nikal ke apni marzi se use kar sakta hai, tumhare account se billing hogi. Hamesha backend se hi LLM API call karna chahiye, frontend backend ko request bheje aur backend LLM ko.

---

**Q7. `usage` field response me kyu important hai?**

Ye batata hai kitne input aur output tokens use hue is request me. Isi se actual cost calculate hoti hai (providers per-token charge karte hain), aur production monitoring/logging ke liye zaroori hai — cost tracking aur rate-limit management dono ke liye.

---

**Q8. Agar tumhe apne QR System jaisa multi-turn chatbot banana ho, conversation history kaha store karoge aur kyu?**

Database ya Redis jaisi in-memory store me (session-based). Har user message aur LLM response save hota hai. Jab agli request aaye, relevant recent history nikal ke `messages` array me bhej dete hain. Redis isliye preferred hai kyunki fast read/write hota hai, session-based data ke liye ideal hai — matches with QR System's approach.

---

**Q9. Temperature parameter ka role API call me kya hota hai (Chapter 1 se connect karo)?**

Temperature controls karta hai model ka next-token selection kitna deterministic ya random hoga. API call me ek parameter ke roop me pass hota hai — low value (0-0.3) factual/consistent tasks ke liye, high value (0.7+) creative tasks ke liye. Default temperature na set karne pe provider ka default value use hota hai (usually 1.0).

---

**Q10. Kya har naye request pe system prompt bhejna zaroori hai?**

Haan, jab tak provider koi explicit caching/session mechanism na de. Kyunki LLM stateless hai, agar system prompt nahi bheja gaya, model ko uske behavior instructions pata nahi chalenge us specific request ke liye — wo generic default behavior pe fall back kar sakta hai.