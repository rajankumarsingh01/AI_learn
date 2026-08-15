# Phase 1 → Chapter 1: How LLMs Work — Interview Q&A

---

**Q1. LLM actually kaam kaise karta hai, simple words me samjhao?**

LLM ek neural network hai jo "next token predict karo" ke task pe train hua hai. Input text tokens me todta hai, unhe numbers (embeddings) me convert karta hai, attention layers se process karta hai, aur output me next token ki probability nikalta hai. Ye process ek-ek token karke repeat hota hai jab tak response complete na ho.

---

**Q2. Token kya hota hai aur word se kaise different hai?**

Token text ka chhota piece hota hai — poora word nahi, roughly ¾ word ke barabar. Ek word multiple tokens me bhi tut sakta hai (jaise "learning" → "learn" + "ing"). Models tokens pe kaam karte hain kyunki fixed vocabulary se efficiently har possible word represent kar sakte hain, chahe wo rare ho ya naya.

---

**Q3. Context window kya hota hai aur production system me isse kaise handle karte ho?**

Context window ek single request me model kitne tokens (input + output milakar) process kar sakta hai, uski limit hai. Production me — jaise mera QR System project — poori conversation history database/Redis me store karte hain, aur har request pe sirf recent/relevant part context window me bhejte hain, taaki limit exceed na ho.

---

**Q4. Temperature parameter kya control karta hai?**

Temperature next-token selection me randomness control karta hai. Low temperature (0-0.3) = deterministic, hamesha highest-probability token chunta hai — factual/code tasks ke liye best. High temperature (1-2) = zyada random, low-probability tokens bhi chun sakta hai — creative writing ke liye best.

---

**Q5. Production RAG system me temperature kya rakhoge aur kyu?**

Low, jaise 0-0.3. Kyunki RAG system me factual, consistent, retrieved-data-based answers chahiye — creativity/randomness nahi. High temperature hallucination ka risk badhata hai.

---

**Q6. Kya LLM ko poori conversation history hamesha yaad rehti hai?**

Nahi. LLM ki koi persistent memory nahi hoti — sirf current context window ke andar jo tokens diye gaye hain, unhi tak "aware" rehta hai. Context window ke bahar ka data model automatically retain nahi karta, jab tak explicitly re-send na kiya jaye.

---

**Q7. Kya base LLM real-time internet se information nikal sakta hai?**

Nahi, bina tools ke nahi. Base LLM sirf training data pe seekhe hue patterns se predict karta hai — uska training data ek fixed cutoff tak ka hota hai. Real-time/current info ke liye tool calling ya RAG (retrieval) explicitly integrate karna padta hai.

---

**Q8. Agar temperature 0 rakh do, kya LLM guaranteed sahi jawab dega?**

Nahi. Temperature sirf randomness control karta hai, factual correctness guarantee nahi karta. Model phir bhi hallucinate kar sakta hai agar uske training data me galat pattern seekha gaya ho ya query training data se bahar ka ho. Isiliye RAG/grounding zaroori hai factual accuracy ke liye.

---

**Q9. Hinglish ya Hindi text LLM ke liye English se zyada expensive (tokens ke hisaab se) kyu hota hai?**

Kyunki zyadatar models primarily English text pe zyada train hote hain, unka tokenizer English words ko efficiently chhote token-count me tod leta hai. Devanagari script ya mixed Hinglish inefficiently tokenize hoti hai — same meaning ke liye zyada tokens lagte hain, matlab zyada cost aur context window jaldi bharta hai.

---

**Q10. "Next-token prediction" ka poora loop ek example se samjhao.**

Input "Capital of France is" diya. Model tokenize karta hai, phir next token ki probability nikalta hai (" Paris" = 85% chance). Highest probability wala token select hota hai, wo naya token input me add ho jata hai ("Capital of France is Paris"), aur phir agla token predict hota hai is naye extended input pe. Ye loop chalta hai jab tak model stop-signal na de ya max-token limit na aa jaye.