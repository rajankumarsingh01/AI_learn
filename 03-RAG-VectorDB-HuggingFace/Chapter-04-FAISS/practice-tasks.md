# Phase 3 - Chapter 4: Practice Tasks - FAISS

## Task 1: Basic Flat Index with L2 Distance

`all-MiniLM-L6-v2` se 6 documents ka embedding generate karo (koi bhi topic). `IndexFlatL2` banao, embeddings add karo, aur ek query chalao — top 3 results print karo (documents ke text ke saath, index number se manually map karke).

## Task 2: Cosine Similarity Setup with IndexFlatIP

Same 6 documents ke embeddings ko normalize karo (`faiss.normalize_L2`), `IndexFlatIP` index banao, aur wahi query dobara chalao. Compare karo — Task 1 (L2) aur Task 2 (Cosine/IP) ke results me ordering same hai ya different? Comment me likho apna observation.

## Task 3: IVF Index Experiment

Kam se kam 200 dummy documents banao (random sentences generate kar sakte ho, ya kisi text dataset se). `IndexIVFFlat` banao (`nlist=10`), train karo, add karo, aur query chalao do different `nprobe` values ke saath (jaise `nprobe=1` aur `nprobe=5`). Compare karo results aur note karo (comment me) — kya farak dikha speed aur result quality me.

## Task 4: Apne QR System Me Apply Karo

Chapter 3 me tumne QR System ka menu search ChromaDB se banaya tha. Ab isi cheez ko FAISS se implement karo, taaki dono approaches ka practical farak samajh aaye:

- QR System ke menu items (Chapter 3 wale hi, ya naye 15-20 items) ka FAISS `IndexFlatIP` (normalized, cosine-equivalent) index banao
- Ek separate Python dictionary/list banao jo FAISS index-number → menu item (naam, description, metadata) map kare — kyunki FAISS khud ye nahi karta
- Ek function likho `search_menu_faiss(user_query, top_k=3)` jo query embed kare, FAISS se search kare, aur mapped menu items return kare
- Socho aur likho (comment ke form me): Chapter 3 ke ChromaDB implementation aur is FAISS implementation ko compare karo — code complexity, metadata filtering ki availability, aur "agar menu items 100 se badh ke 1 lakh ho jaayein" scenario me kaunsa approach better scale karega, in teeno angles se apna analysis likho