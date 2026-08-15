# Phase 3 - Chapter 3: Practice Tasks - Vector Databases (ChromaDB)

## Task 1: Basic Collection Setup & Insertion

`PersistentClient` use karke ek ChromaDB collection banao (`path="./chroma_db"`). Usme kam se kam 8 documents add karo (koi bhi topic — movies, books, ya food items), har ek ke saath ek unique `id` aur ek `metadata` field (jaise category ya type). Sentence-Transformers `all-MiniLM-L6-v2` se embeddings generate karke pass karo (Option B approach jaisa notes.md me diya hai).

## Task 2: Query with Metadata Filtering

Ek function likho `search_with_filter(query, filter_dict, top_n=3)` jo:
- Query ko embed kare
- `where` clause me `filter_dict` pass kare
- Top `top_n` results return kare (documents + distances dono ke saath)

Test karo isi function ko do tarike se — (1) filter ke saath, (2) filter ke bina — aur observe karo results kaise change hote hain.

## Task 3: Distance Metric Comparison

Do collections banao — ek `hnsw:space="cosine"` ke saath, ek default (`l2`) ke saath. Same documents dono me add karo, same query chalao, aur compare karo ki distance values aur result ordering me kya farak aata hai. Note karo (comment me) — kaunsa metric tumhare use case ke liye better lagta hai aur kyu.

## Task 4: Apne QR System Me Apply Karo

Tumhare QR Food Ordering System me abhi chatbot function calling aur Redis session memory use karta hai, lekin menu search abhi (Task 4, Chapter 2 me) sirf in-memory cosine similarity se ho raha tha. Ab isko production-ready banao:

- Apne QR System ke menu items (naam, description, category, veg/non-veg, price) ka ek ChromaDB collection banao — `PersistentClient` use karke, taaki restart pe data na khoye
- Ek function likho `search_menu_v2(user_query, veg_only=False)` jo semantic search karta ho AND `veg_only=True` hone par metadata filter bhi apply kare
- Socho aur likho (comment ke form me): ye ChromaDB-based approach Chapter 2 ke manual cosine-similarity-loop approach se production me kaise better hai — specifically latency aur scalability ke angle se jab menu items 10 se badh ke 500+ ho jaayein
- Bonus (optional): socho ki Redis session memory (jo tumhare chatbot me already hai) aur ChromaDB dono ka role RAG pipeline me kaise alag-alag hai — Redis kis cheez ke liye hai, ChromaDB kis cheez ke liye