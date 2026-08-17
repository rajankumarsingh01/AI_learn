# Phase 3 - Chapter 5: Interview Q&A - Chunking Strategies

**Q1: Chunking RAG pipeline me kyu zaroori hai — direct poora document embed kyu nahi kar dete?**
A: Embedding models ki maximum token limit hoti hai (typically 256-512 tokens) — isse zyada lamba text silently truncate ho jaata hai. Iske alawa, poora document ek chunk hone se retrieval imprecise ho jaata hai — query match hone pe pura document context me jaata hai jisme sirf ek chhota relevant part hota hai aur baaki irrelevant noise, jo LLM ke response quality aur token cost dono ko affect karta hai.

**Q2: Fixed-size chunking aur recursive/semantic chunking me kya farak hai?**
A: Fixed-size chunking blindly character/token count pe text todta hai — kisi sentence ke beech me bhi boundary aa sakti hai, jo meaning distort karta hai. Recursive chunking hierarchy follow karta hai (pehle paragraph breaks try karo, phir sentence, phir word) — isse chunks naturally meaningful semantic boundaries pe bante hain jab tak possible ho, jo retrieval quality improve karta hai.

**Q3: Chunk overlap kya hai aur ye kyu use karte hain?**
A: Overlap ka matlab hai consecutive chunks ke beech kuch common text rakhna (typically 10-20% of chunk size). Isse har chunk apne aap me contextually self-sufficient rehta hai — bina overlap ke, chunk boundary pe context "kat" jaata hai aur adjacent information disconnected lagti hai jab wo chunk akela retrieve hota hai.

**Q4: Chunk size choose karte time kya trade-offs consider karoge?**
A: Chhote chunks precise, focused retrieval dete hain lekin context incomplete ho sakta hai aur storage/chunk-count badh jaata hai. Bade chunks zyada context per chunk dete hain lekin irrelevant information mix ho sakti hai aur embedding quality degrade ho sakti hai (multiple topics ek chunk me average out ho jaate hain). Typically 300-500 tokens with 10-15% overlap ek practical starting point hai, jo use-case ke hisaab se tune hota hai.

**Q5: Agar chunk size embedding model ki max token limit se bada ho, to kya hoga?**
A: Embedding model text ko silently truncate kar dega — koi error nahi aayega, lekin chunk ka baaki hissa embedding computation me include hi nahi hoga. Ye ek silent bug hai jo debug karna mushkil hota hai kyunki system crash nahi karta, bas retrieval quality chupke se kharab ho jaati hai. Isliye chunk size ko embedding model ke max sequence length ke andar rakhna critical hai.

**Q6: Different document types (jaise code, legal docs, chat logs) ke liye same chunking strategy use kar sakte hain kya?**
A: Nahi, ye best practice nahi hai. Code documentation me function/class boundaries natural split points hain. Legal documents me clauses/sections. Chat conversations me speaker turns. Ek generic fixed-size approach in structures ko ignore karta hai. Production RAG systems content-type-aware chunking use karte hain — jaise code ke liye AST-based splitting, ya markdown documents ke liye heading-based splitting.

**Q7: Chunking ke baad metadata track karna kyu important hai?**
A: Sirf raw text chunk karke store karna kaafi nahi hai — source document, chunk position (jaise "document X ka chunk 3 of 10"), aur original context ka reference rakhna zaroori hai. Isse retrieval ke baad user/system ko trace karne me madad milti hai ki information kis document/section se aayi, jo transparency aur debugging dono ke liye important hai — especially agar multiple source documents hon.

**Q8: Sentence-based chunking me kya limitation ho sakti hai?**
A: Sentence boundaries respect karne se semantic meaning to preserve hota hai, lekin sentences ki length bahut vary kar sakti hai — kuch sentences chhote hote hain (5 words), kuch bahut lambe (50+ words). Isse chunk sizes inconsistent ho jaate hain, jo downstream embedding/retrieval consistency ko affect kar sakta hai. Isliye typically sentence-based approach ko max_chunk_size constraint ke saath combine karte hain.

**Q9: "Semantic chunking" (embedding-based chunking) kya hota hai, aur ye traditional methods se kaise alag hai?**
A: Semantic chunking me consecutive sentences ke embeddings calculate karke unki similarity check ki jaati hai — jahan similarity drop hoti hai (topic change ho raha ho), wahi natural chunk boundary maani jaati hai. Ye traditional character/sentence-count based methods se zyada intelligent hai kyunki ye actual content ke meaning ke basis pe todta hai, na ki arbitrary length ke basis pe — lekin computationally zyada expensive hai kyunki chunking se pehle hi embeddings calculate karne padte hain.

**Q10: Rajan ke QR System context me — agar tum apna poora menu + policies document (5 pages) RAG me daalna chaho, to chunking approach kya hoga?**
A: Recursive character splitting use karunga jisme separators hierarchy ho — pehle section headers (jaise "## Refund Policy") pe split, phir paragraphs, phir sentences agar zaroorat pade. Chunk size ~300-400 characters rakhunga with ~50 character overlap, aur har chunk ke saath metadata attach karunga (jaise `section: "refund_policy"` ya `section: "menu_item"`) taaki Chapter 3 ke ChromaDB metadata filtering ka use karke query time pe relevant section se hi results filter kar sakun.