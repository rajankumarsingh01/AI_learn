# Phase 3 - Chapter 4: Interview Q&A - FAISS

**Q1: FAISS aur ChromaDB me fundamental difference kya hai?**
A: ChromaDB ek complete vector database hai — storage, indexing, metadata filtering, persistence sab built-in deta hai. FAISS sirf ek indexing/similarity-search library hai — ye bahut fast search deta hai lekin persistence, metadata management, aur document-mapping khud implement karne padte hain. FAISS "engine" hai, ChromaDB "poori car" hai.

**Q2: `IndexFlatL2` aur `IndexFlatIP` me kya farak hai, aur kab kaunsa use karoge?**
A: `IndexFlatL2` Euclidean (L2) distance use karta hai — lower value better match. `IndexFlatIP` Inner Product use karta hai — higher value better match, aur ye cosine similarity ke equivalent tabhi hota hai jab vectors normalized hon (`faiss.normalize_L2()`). Semantic search (RAG) me generally cosine similarity better result deta hai magnitude-independent hone ki wajah se, isliye normalized vectors ke saath `IndexFlatIP` preferred hai.

**Q3: IVF Index kya hai aur ye Flat Index se kaise better hai large scale pe?**
A: IVF (Inverted File Index) dataset ko `nlist` clusters me divide karta hai (k-means jaisa). Query aane pe, sirf kuch relevant clusters (jitna `nprobe` specify kare) search kiye jaate hain, poora dataset nahi — isse search bahut fast hoti hai bade datasets pe, thoda accuracy trade-off ke saath (approximate search, exact nahi).

**Q4: `nprobe` parameter kya control karta hai?**
A: `nprobe` batata hai ki query time pe kitne clusters check kiye jayein. Kam `nprobe` = fast lekin kam accurate (kuch relevant results miss ho sakte hain agar wo dusre cluster me hon). Zyada `nprobe` = slow lekin zyada accurate. Ye classic speed-vs-accuracy trade-off hai jo production tuning me important hota hai.

**Q5: FAISS index ko train() karna kab zaroori hai, aur kyu?**
A: `IndexFlatL2`/`IndexFlatIP` jaise simple indexes ko train() ki zaroorat nahi (ye brute-force hain). Lekin `IndexIVFFlat` jaise approximate indexes ko train() karna mandatory hai — kyunki train() step clusters (centroids) seekhta hai data se, jo baad me search ke time use hote hain. Bina train() kiye `add()` karoge to error aayega.

**Q6: Agar FAISS embeddings float64 me pass kiye jaayein to kya hoga?**
A: FAISS internally C++ optimized code use karta hai jo strictly `float32` numpy arrays expect karta hai. `float64` ya Python list pass karne pe ya to type error aayega ya unexpected/incorrect behavior hoga. Isliye hamesha `.astype('float32')` explicitly apply karna chahiye embeddings pe.

**Q7: FAISS me metadata filtering (jaise "sirf veg items search karo") kaise achieve karoge, jab ye built-in feature nahi hai?**
A: Do common approaches hain — (1) Pre-filtering: pehle metadata ke basis pe candidate documents ka subset nikalo (Python/pandas se), phir sirf unka index banao aur search karo. (2) Post-filtering: pura similarity search karo (zyada `k` value ke saath), phir results ko metadata ke basis pe filter karo. Dono approaches ka trade-off hai — pre-filtering zyada efficient hai agar filter selective ho, post-filtering simpler hai implement karne me.

**Q8: FAISS index ko disk pe persist kaise karte ho, aur documents/metadata ka kya karte ho?**
A: `faiss.write_index(index, "path.faiss")` se index save hota hai aur `faiss.read_index()` se load hota hai — lekin ye sirf vectors save karta hai. Original documents aur metadata ko separately manage karna padta hai — typically ek JSON file, pickle file, ya lightweight DB (SQLite) me, jisme FAISS index-number → document mapping store ho.

**Q9: Interview me pooche jaane wala classic question — "10 lakh vectors pe real-time similarity search kaise design karoge?"**
A: Flat Index (brute-force) 10 lakh vectors pe slow ho jayega real-time use case ke liye. IVF Index use karunga (ya HNSW-based approach) jo approximate nearest neighbor search karta hai — clusters banake sirf relevant clusters search karta hai. `nlist` aur `nprobe` ko dataset size aur latency requirements ke hisaab se tune karunga — trade-off explain karna important hai (speed vs recall accuracy).

**Q10: Chhote projects (jaise Rajan ke QR System, ~50-100 menu items) ke liye FAISS use karna chahiye ya ChromaDB?**
A: Chhote scale pe (sau-do sau items) ChromaDB better choice hai — kyunki ye persistence, metadata filtering built-in deta hai, setup effort kam hai, aur is scale pe FAISS ki extra raw speed noticeable benefit nahi degi. FAISS tab justify hota hai jab dataset lakhon-crodon vectors ka ho, ya jab custom indexing control chahiye ho jo ek managed vector DB nahi deta.