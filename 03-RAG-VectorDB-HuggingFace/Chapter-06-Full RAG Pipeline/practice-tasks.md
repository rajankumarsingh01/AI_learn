# Phase 3 - Chapter 6: Practice Tasks - Full RAG Pipeline

## Task 1: Build Complete Indexing Pipeline

`index_document()` function use karke 2-3 alag documents (dummy text, kam se kam 500 characters har ek) index karo apne ChromaDB collection me. `collection.count()` se verify karo ki saare chunks properly stored hue. Print karo total chunks per document.

## Task 2: Build Retrieval + Prompt Construction

`retrieve_context()` aur `build_prompt()` functions use karke 3 alag queries chalao apne indexed documents pe. Har query ke liye print karo — retrieved chunks, unke sources, aur final constructed prompt (jo LLM ko jaayega). Verify karo ki prompt properly formatted hai aur context clearly included hai.

## Task 3: Retrieval Accuracy Evaluation

`evaluate_retrieval()` jaisa function use karke kam se kam 5 test cases banao (query + expected source pairs) apne indexed documents ke basis pe. Accuracy calculate karo. Agar accuracy 100% nahi aati, to analyze karo (comment me likho) — kaunsi queries fail hui aur kyu (chunk size issue, ya query phrasing issue, ya kuch aur).

## Task 4: Apne QR System Me Full RAG Add Karo (Capstone Task)

Ye Phase 3 ka capstone task hai — sab kuch ek saath jodo apne QR Food Ordering System ke context me:

- Ek complete "Knowledge Base" document banao QR System ke liye — menu items, policies (refund, delivery), FAQs (dummy content chalega, kam se kam 800-1000 words, headers ke saath)
- Isse index karo (chunking + embedding + ChromaDB storage) using `index_document()`
- `rag_query()` function complete karo jisme actual LLM API call bhi ho (Phase 1 Chapter 2 ka Anthropic/OpenAI API knowledge use karo) — taaki tumhe real LLM response mile, na sirf constructed prompt
- Kam se kam 5 alag queries test karo (mix of menu questions, policy questions, aur ek aisa question jiska answer document me hai hi nahi — verify karo ki system "information available nahi hai" jaisa bolta hai, hallucinate nahi karta)
- Socho aur likho (comment ke form me): ye RAG feature tumhare existing QR System chatbot (jo function calling + Redis session memory use karta hai) me kaise integrate hoga — specifically, chatbot kaise decide karega ki kab function calling use karna hai (order place karna) aur kab RAG use karna hai (menu/policy question answer karna)? Ye decision-routing problem ka apna solution design likho (2-3 sentences, high-level)