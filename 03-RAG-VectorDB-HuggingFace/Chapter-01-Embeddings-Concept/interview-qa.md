# Phase 3 → Chapter 1: Embeddings Concept — Interview Q&A

---

**Q1. Embedding kya hota hai, simple words me samjhao?**

Embedding ek text (word, sentence, ya paragraph) ko numbers ki ek fixed-length list (vector) me convert karne ka tareeka hai, is tarah ki similar meaning wale texts ke vectors mathematically close hote hain. Ye text ke "meaning" ko numerically represent karta hai.

---

**Q2. Embedding model aur LLM me kya difference hai?**

Dono alag purpose ke models hain. Embedding model text ko vector (numbers) me convert karta hai, koi text generate nahi karta. LLM text generate karta hai (next-token prediction se). RAG pipelines me dono use hote hain — embedding model retrieval ke liye, LLM final response generation ke liye.

---

**Q3. Cosine similarity kya hai aur ye kya measure karta hai?**

Ye ek method hai do vectors ke beech "angle" measure karne ka (magnitude nahi, direction), taaki decide kiya ja sake do texts kitne semantically similar hain. Score -1 se 1 ke beech hota hai — 1.0 matlab same meaning, 0.0 matlab koi relation nahi.

---

**Q4. Semantic search aur keyword search me kya difference hai, example ke saath samjhao?**

Keyword search exact words match karta hai — "tasty" search karoge toh sirf wahi documents milenge jisme literally "tasty" word ho. Semantic search embeddings use karke meaning match karta hai — "paneer tikka achha hai" query se "paneer tikka delicious hai" wala result bhi mil jayega, chahe koi common exact word na ho, kyunki dono ka meaning similar hai.

---

**Q5. Embedding dimensions kya matlab rakhte hain (jaise 384 vs 1536)?**

Ye batata hai vector me kitne numbers hain — zyada dimensions zyada "nuance"/detail capture kar sakte hain meaning ka, lekin zyada storage aur compute bhi chahiye. Hardware-constrained setups ke liye chhote dimension models (384-768) practical choice hote hain, bina major accuracy loss ke most use-cases me.

---

**Q6. RAG pipeline me embeddings ka role kya hai, high-level flow batao?**

Documents ko chunks me todkar unke embeddings generate karke vector database me store karte hain. User query aane par uska bhi embedding banate hain, aur us se sabse similar chunks vector DB se retrieve karte hain (cosine similarity se). Wo relevant chunks LLM ko context ke roop me diye jate hain taaki wo grounded, accurate answer de sake.

---

**Q7. Agar do sentences me koi common exact word na ho, phir bhi unka cosine similarity high kaise aa sakta hai?**

Kyunki embeddings word-matching pe based nahi hote, meaning/semantic representation pe based hote hain. Jaise "paneer tikka is tasty" aur "grilled paneer cubes are delicious" — words alag hain but meaning similar hai, isliye embedding model dono ko close vectors assign karega, high similarity aayegi.

---

**Q8. Cosine similarity score ko "percentage accuracy" samajhna kyu galat hai?**

Kyunki ye ek relative closeness measure hai (do specific texts kitne similar hain ek dusre se), absolute correctness ya confidence ka measure nahi. Ek high similarity score sirf ye batata hai texts semantically close hain, ye nahi batata ki information factually correct hai.

---

**Q9. Chhote embedding model (384 dimensions) use karne ka trade-off kya hai bade model (1536 dimensions) ke against?**

Chhote model kam storage/compute lete hain, faster hote hain — hardware-constrained setups (jaise 8GB RAM) ke liye practical hain. Trade-off ye hai ki wo kabhi-kabhi subtle/nuanced semantic differences utni precisely capture nahi kar pate jitna bade models karte hain — lekin most practical use-cases (FAQ search, notes search) ke liye difference minor hota hai.

---

**Q10. Kya embeddings sirf English text ke liye kaam karte hain, ya Hindi/Hinglish text ke liye bhi?**

Kaam karte hain, lekin quality model ke training data pe depend karti hai. Multilingual embedding models (jo specifically Hindi/mixed-language data pe bhi train hue hon) better perform karte hain non-English text pe compared to primarily-English-trained models. Model choose karte waqt ye consideration important hai agar Hinglish/Hindi content handle karna ho.