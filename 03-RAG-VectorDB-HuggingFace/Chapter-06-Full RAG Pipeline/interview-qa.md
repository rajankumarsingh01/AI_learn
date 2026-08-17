# Phase 3 - Chapter 6: Interview Q&A - Full RAG Pipeline

**Q1: RAG pipeline ke do main phases kya hain, aur inme kya farak hai?**
A: Indexing Phase (ek baar ya jab data update ho) — document ko chunk karke, embed karke, vector DB me store karte hain. Query Phase (har user request pe) — user query ko embed karke, vector DB se relevant chunks retrieve karte hain, aur unhe LLM prompt me context ke roop me daal ke final answer generate karte hain. Indexing phase "prepare" karta hai, query phase "serve" karta hai.

**Q2: RAG kya problem solve karta hai jo plain LLM API call nahi kar sakti?**
A: Plain LLM sirf apni training data pe based answer deta hai — jo outdated ho sakta hai ya specific/private information (jaise company ke internal docs) contain nahi karta. RAG LLM ko real-time, specific, aur verifiable context provide karta hai retrieval ke through, jisse hallucination kam hota hai aur answers grounded/up-to-date hote hain — bina model ko retrain kiye.

**Q3: Agar RAG system galat answer de raha hai, to debug kaise karoge — kaunsa phase check karoge pehle?**
A: Pehle retrieval phase isolate karke check karunga — `retrieve_context()` ka output dekhunga ki relevant chunks aa rahe hain ya nahi. Agar retrieval sahi hai (relevant chunks aa rahe hain) lekin final answer galat hai, to problem prompt construction ya LLM ke instruction-following me hai. Agar retrieval hi galat chunks laa raha hai, to chunking strategy ya embedding model check karunga. Ye separation of concerns RAG debugging ka core skill hai.

**Q4: Prompt me "sirf context se answer do" jaisa explicit instruction dena kyu critical hai?**
A: Bina explicit instruction ke, LLM apni training data (parametric knowledge) aur diye gaye context dono ko mix kar sakta hai — jo RAG ka poora purpose (grounded, verifiable answers) defeat karta hai. Explicit instruction LLM ko constrain karta hai ki wo primarily/sirf provided context use kare, aur agar context me answer nahi hai to saaf bataye "information available nahi hai" — jo hallucination significantly reduce karta hai.

**Q5: RAG pipeline me `top_k` ka value kaise decide karoge?**
A: `top_k` trade-off hai — kam value (2-3) precise but incomplete context de sakta hai, zyada value (10+) zyada context lekin noise/irrelevant info bhi la sakta hai jo LLM ko confuse kar sakta hai aur token cost badhata hai. Typically 3-5 se start karke, evaluation (retrieval accuracy testing) ke basis pe tune karta hoon. Advanced systems query complexity ke hisaab se dynamically adjust bhi karte hain.

**Q6: RAG system ki "retrieval accuracy" kaise measure karoge?**
A: Test cases banata hoon jisme known queries aur unke expected/correct source documents defined hon. Har query ko retrieve karke check karta hoon ki expected source retrieved results me present hai ya nahi (top-k me). Correct matches ka percentage nikaal ke accuracy measure hoti hai. Production systems me isse aage precision, recall, aur relevance scoring jaise sophisticated metrics bhi use hote hain (Phase 7 Evals me cover hoga).

**Q7: Agar indexing phase aur query phase alag embedding models use karein, to system me kya symptom dikhega?**
A: Retrieval consistently poor/irrelevant results dega — kabhi-kabhi dimension mismatch error bhi aa sakta hai agar models ke output dimensions different hon. Ye ek subtle bug hai jo system crash nahi karega (agar dimensions match ho jaayen coincidentally) lekin silently bad results dega, jo debug karna time-consuming ho sakta hai agar developer ko pata na ho ki model consistency check karni hai.

**Q8: LangChain jaisa framework RAG pipeline ko kaise simplify karta hai jo humne manually banaya?**
A: LangChain chunking, embedding, vector store integration, aur retrieval-generation chaining ko pre-built abstractions me wrap karta hai — jaise `TextSplitter`, `VectorStore`, aur `RetrievalQA` chain classes. Manual implementation samajhne ka fayda ye hai ki jab LangChain "andar se" kaam kaise karta hai, wo clear hota hai — jisse debugging aur customization dono easier hote hain jab framework ka default behavior kaafi na ho.

**Q9: RAG pipeline me "hallucination" ka risk pura khatam ho jaata hai kya?**
A: Nahi, RAG hallucination ka risk significantly REDUCE karta hai, khatam nahi karta. LLM phir bhi retrieved context ko galat interpret kar sakta hai, ya agar retrieval hi weak/irrelevant chunks laaye, to LLM ke paas galat foundation hoga answer banane ke liye. Isliye retrieval quality aur prompt engineering dono equally important hain — RAG ek mitigation strategy hai, guarantee nahi.

**Q10: Rajan apne QR System me RAG add karna chahta hai (menu/policy questions ke liye) — high-level design kaise explain karega interview me?**
A: Indexing phase me — menu items aur policies documents ko chunk karunga (Ch5 approach), Sentence-Transformers se embed karunga (Ch2), ChromaDB me metadata (category, section) ke saath store karunga (Ch3). Query phase me — user ka natural language question (jaise "kya aapke paas jain food hai") embed karke ChromaDB se top-3 relevant chunks retrieve karunga, phir un chunks ko context ke roop me apne existing chatbot ke LLM call (jo already function calling use karta hai, Phase 2 yaad karo) me inject karunga. Ye RAG capability chatbot ke existing function-calling architecture ke saath complement karegi — function calling structured actions (order place karna) ke liye, RAG unstructured knowledge queries (menu/policy questions) ke liye.