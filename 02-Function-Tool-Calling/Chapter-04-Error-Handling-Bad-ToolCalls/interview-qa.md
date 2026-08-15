# Phase 2 → Chapter 4: Error Handling — Bad Tool Calls — Interview Q&A

---

**Q1. Tool calling me kaunse teen main tarah ki failures ho sakti hain?**

(1) LLM ne galat tool choose kiya (schema ambiguity ki wajah se), (2) LLM ne sahi tool choose kiya lekin arguments missing/invalid the, (3) Tool khud execute karte waqt fail ho gaya (database down, network error, external API failure).

---

**Q2. Agar LLM missing arguments ke saath tool call kare (jaise order_id na diya), production code me isse kaise handle karoge?**

Backend validation layer check karega ki required argument missing hai, aur function execute karne ki bajaye ek error result return karega jaise `{success: false, error: "order_id is missing"}`. Ye result LLM ko wapas jata hai, jisse LLM khud user se follow-up sawal poochta hai — conversational recovery hoti hai, crash nahi hota.

---

**Q3. Raw error/stack trace LLM ko bhejna kyu problematic hai?**

Do reasons se: (1) Security risk — internal system details (database structure, file paths, etc.) leak ho sakti hain, (2) LLM confusing/overly-technical response bana sakta hai user ke liye jo unhelpful hoga. Isliye hamesha clean, user-friendly error message wrap karke bhejna chahiye.

---

**Q4. Kaunse error types pe retry karna chahiye, aur kaunse pe nahi?**

Network timeouts aur temporary server errors (5xx) pe retry karna chahiye, exponential backoff ke saath. Missing/invalid arguments (LLM ki galti) ya business logic failures (jaise "order already delivered") pe retry NAHI karna chahiye — inme retry se same result aayega, iske bajaye LLM ko clarification maangne dena chahiye ya user ko clear reason batana chahiye.

---

**Q5. Backend validation zaroori kyu hai jab LLM already schema follow kar raha hai?**

Kyunki LLM ka schema-following usually achha hota hai lekin 100% guaranteed nahi — especially bina strict mode ke ya edge cases me. Backend validation ek safety-net layer hai jo bugs aur potential security issues prevent karti hai, sirf LLM pe blindly trust nahi karte.

---

**Q6. Tool execution fail hone pe (jaise database error) response kaisa design karoge?**

Try-catch me wrap karke, agar error aaye toh raw exception ki jagah ek clean message return karenge jaise `{success: false, error: "Unable to process right now, please try again"}`. Ye LLM ko clear signal deta hai operation fail hua, bina internal details expose kiye.

---

**Q7. Agar tool call ka result LLM ko explicitly success/failure signal nahi diya jaye, kya risk hai?**

LLM assume kar sakta hai operation successful hui hai (chahe actually fail hui ho), aur galat/misleading final response user ko de sakta hai — jaise "aapka order cancel ho gaya" bolna jabki cancellation actually fail hui thi. Isliye explicit `success: boolean` field hona zaroori hai.

---

**Q8. Apne QR System me ek real error scenario batao aur ye kaise handle hoga.**

Agar user "wo cheez add karo" bole bina specific item name diye, `add_item_to_cart` tool call unclear/empty `item_name` ke saath aayega. Backend validation ise catch karegi, error result return karegi, aur LLM user se clarification maangega — "kaunsi item add karni hai, please batayein."

---

**Q9. Business logic failure (jaise "order already delivered, cannot cancel") aur transient error (jaise network timeout) me handling approach kaise different hoga?**

Business logic failure permanent hai is specific attempt ke liye — retry se kuch nahi badlega, LLM ko clear reason batana chahiye user ko. Transient error temporary hai — exponential backoff ke saath retry karna sahi approach hai, kyunki agla attempt succeed ho sakta hai.

---

**Q10. Validation layer kaha implement karni chahiye — LLM-facing tool schema me ya backend function ke andar?**

Dono jagah — schema level pe (types, required fields, enums) pehli layer of defense hai jo LLM ko guide karti hai, lekin backend function ke andar bhi explicit validation honi chahiye kyunki schema-following guaranteed nahi hoti. Backend validation final safety net hai before actual execution (database write, external API call, etc.).