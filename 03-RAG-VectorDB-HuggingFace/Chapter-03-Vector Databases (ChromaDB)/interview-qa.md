# Phase 3 - Chapter 3: Interview Q&A - Vector Databases (ChromaDB)

**Q1: Vector Database normal database (SQL/MongoDB) se kaise different hai?**
A: Normal database exact match ya range queries ke liye optimized hai (WHERE naam = 'X'). Vector DB high-dimensional vectors ke beech "nearest neighbor" similarity search ke liye optimized hai — ye approximate nearest neighbor (ANN) algorithms jaise HNSW use karta hai taaki milliseconds me relevant results mile, chahe lakhon vectors ho. Traditional indexing (B-tree) high-dimensional similarity search ke liye efficient nahi hota.

**Q2: HNSW algorithm kya hai, high-level me samjhao.**
A: HNSW (Hierarchical Navigable Small World) ek graph-based indexing algorithm hai jo vectors ko multiple layers me organize karta hai — top layer sparse hota hai (lambi jumps ke liye), bottom layer dense hota hai (fine-grained search ke liye). Query aane pe search top layer se start hoti hai aur gradually neeche aati hai, har layer pe closest node dhoondte hue — isse brute-force O(n) comparison ki jagah approximately O(log n) time me result milta hai.

**Q3: PersistentClient aur in-memory Client me kya farak hai, aur production me kya use karoge?**
A: `chromadb.Client()` data ko sirf RAM me rakhta hai — process khatam hote hi data gayab ho jaata hai. `PersistentClient(path=...)` data ko disk pe SQLite-based storage me save karta hai, jo restart ke baad bhi persist rehta hai. Production me hamesha `PersistentClient` use karunga, kyunki embeddings baar-baar regenerate karna expensive aur unnecessary hai.

**Q4: Distance aur Similarity me confusion kyu hoti hai, aur ChromaDB me ye kaise interpret karte hain?**
A: `distances` field me LOWER value ka matlab MORE similar hota hai (jaise Euclidean/L2 distance me 0 ka matlab identical vectors). Beginners isko similarity score samajh ke ulta interpret kar dete hain. Agar cosine similarity chahiye (0-1 scale, higher = more similar), to collection banate time explicitly `metadata={"hnsw:space": "cosine"}` set karna padta hai, aur phir bhi ChromaDB "cosine distance" return karta hai (1 - cosine similarity), similarity nahi.

**Q5: Metadata filtering (`where` clause) RAG system me kyu important hai?**
A: Pure semantic search kabhi-kabhi irrelevant results de sakta hai (jaise non-veg item match ho jaana jab user ne "vegetarian" bola ho, agar embedding model ne exact word match na kiya ho). Metadata filtering se hum structured constraints (category, date, user-role, veg/non-veg) enforce kar sakte hain similarity search ke saath — ye hybrid search approach real-world RAG systems me accuracy significantly improve karta hai.

**Q6: Agar embedding generate karne wala model add() aur query() me alag ho, to kya hoga?**
A: Dono embeddings different vector spaces me honge (different dimensions ya different semantic representations), isliye similarity comparison meaningless ho jayega — ya to dimension mismatch error aayega, ya agar dimensions same bhi hue (coincidentally), to distances garbage honge kyunki dono models ka "semantic understanding" alag hai. Same embedding model consistently use karna (indexing aur querying dono me) mandatory hai.

**Q7: ChromaDB jaisa local vector DB kab use karoge, aur Pinecone/Weaviate jaisa managed cloud vector DB kab?**
A: ChromaDB tab use karunga jab: prototype/development phase ho, dataset size manageable ho (lakhs tak), aur infra simplicity chahiye ho (koi separate server maintain nahi karna). Managed cloud vector DB (Pinecone, Weaviate, Qdrant Cloud) tab consider karunga jab: massive scale ho (crores of vectors), high availability/distributed setup chahiye ho, ya team ko infra maintain karne ka bandwidth na ho. Startups me typically ChromaDB se start karke scale ke saath migrate karte hain.

**Q8: RAG pipeline me Vector DB exact kahan fit hota hai — data flow explain karo.**
A: Do phases hain — (1) **Indexing phase**: Documents ko chunk karo → Sentence-Transformers se embed karo → ChromaDB collection me store karo (embeddings + metadata + original text). (2) **Query phase**: User query aati hai → usi model se embed karo → ChromaDB se `n_results` closest matches retrieve karo (with optional metadata filter) → retrieved documents ko context ke roop me LLM prompt me inject karo → LLM final answer generate karta hai. Vector DB is retrieval step ka core hai.

**Q9: `n_results` ka value zyada ya kam rakhne ka trade-off kya hai?**
A: Kam `n_results` (jaise 3) — fast, kam token cost, lekin agar relevant document top-3 me na aaya to information miss ho sakti hai. Zyada `n_results` (jaise 20) — zyada recall (relevant document milne ke chances zyada), lekin LLM ko unnecessary/noisy context milta hai jo response quality kharab kar sakta hai aur token cost badhata hai. Production me typically `n_results=3-5` rakhte hain, phir zaroorat pade to re-ranking step add karte hain.

**Q10: ChromaDB collection me embeddings store karne ke bajaye har baar real-time embed karna kyu bad practice hai?**
A: Real-time embedding generation (query ke alawa) compute-expensive hai — agar 1000 documents har request pe re-embed kiye jayenge, to latency drastically badhegi (seconds ka overhead) aur CPU/GPU resources waste honge. Vector DB ka poora purpose hi ye hai ki embeddings ek baar generate karke store kiye jaayein (indexing phase), aur sirf incoming query ko real-time embed kiya jaaye — ye architecture decision RAG system ki scalability ke liye fundamental hai.