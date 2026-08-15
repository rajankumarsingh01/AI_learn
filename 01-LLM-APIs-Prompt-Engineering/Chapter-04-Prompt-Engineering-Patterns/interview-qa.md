# Phase 1 → Chapter 4: Prompt Engineering Patterns — Interview Q&A

---

**Q1. Zero-shot aur few-shot prompting me kya difference hai?**

Zero-shot me model ko koi example diye bina seedha task diya jata hai — simple, common tasks ke liye theek hai. Few-shot me 2-3 examples diye jate hain jisse model ko exact output format, tone, aur style samajh aata hai — complex ya domain-specific tasks me consistency ke liye better hai.

---

**Q2. Chain-of-thought (CoT) prompting kya hai aur kab use karna chahiye?**

CoT me model ko explicitly "step by step think karo" bola jata hai, jisse wo directly jump karke answer dene ki bajaye reasoning process follow karta hai. Complex, multi-step logical/mathematical tasks me accuracy significantly improve hoti hai. Simple, straightforward tasks pe CoT force karna sirf latency/token cost badhata hai bina benefit ke.

---

**Q3. Role-based prompting se actual output kaise change hota hai, same model ke saath?**

Same underlying model hone ke bawajood, system prompt me specific persona/role define karne se (jaise "senior security engineer" vs "friendly support assistant") response ka tone, vocabulary, aur focus dramatically change ho jata hai — kyunki model us role ke associated patterns follow karta hai jo usne training data me seekhe hain.

---

**Q4. Production system prompts me kaunsa structure follow karna best practice hai?**

Role → Task → Constraints → Output Format → Examples — is structure me likhna chahiye, random paragraph ki jagah. Ye readable bhi hota hai aur model ke liye follow karna bhi easier hota hai, especially jab multiple constraints ho.

---

**Q5. Negative prompting (kya nahi karna) kyu zaroori hota hai, sirf positive instructions kaafi kyu nahi?**

Kyunki bina explicit constraints ke, model "helpful lagne ki koshish" me galat/fabricated information de sakta hai (hallucination) ya off-topic conversation me chala ja sakta hai. Explicitly "do NOT make up information," "do NOT discuss unrelated topics" jaisi instructions in risks ko kam karti hain.

---

**Q6. Few-shot prompting me examples ka format inconsistent ho toh kya problem hoti hai?**

Model confuse ho jata hai kyunki wo examples se pattern seekhne ki koshish karta hai — agar examples khud hi different format/style follow kar rahe hain, model ko clear signal nahi milta ki actual desired output kaisa hona chahiye, output unpredictable ho sakta hai.

---

**Q7. Apne QR System ke agentic chatbot me tumne prompt engineering kaise apply ki, ya karoge?**

Order extraction ke liye structured instruction pattern use karenge — Role (order-extraction assistant), Task (Hinglish/English se items nikalna), Constraints (sirf explicitly mentioned items, quantity default 1), Output Format (JSON schema), aur few-shot examples Hinglish input ke saath — taaki consistent extraction ho.

---

**Q8. Modern reasoning models (jaise o1, extended thinking mode) me explicit "think step by step" bolna zaroori hai kya?**

Kam zaroori hai — ye models internally hamesha CoT-jaisa reasoning process follow karte hain by default. Lekin normal chat models (non-reasoning) me explicit instruction dena still helpful hai accuracy improve karne ke liye complex tasks pe.

---

**Q9. Vague prompt jaisa "achha jawab do" kyu problematic hai?**

Kyunki ye specific, measurable instruction nahi hai — model ko pata nahi "achha" ka matlab kya hai (lamba? chhota? formal? casual?). Isse output inconsistent aur unpredictable hota hai. Specific instructions ("2-3 sentences me, formal tone") dena better hai.

---

**Q10. Prompt engineering RAG aur agentic systems ka foundation kyu hai?**

Kyunki RAG me retrieval-augmented context ko effectively LLM tak pahunchana (structure kaise karna hai), aur agents me tool-calling decisions guide karna (kab kaunsa tool use kare) — dono hi achhi prompt design pe depend karte hain. Weak prompt engineering se RAG/agent systems unreliable output denge chahe underlying architecture kitni bhi achhi ho.