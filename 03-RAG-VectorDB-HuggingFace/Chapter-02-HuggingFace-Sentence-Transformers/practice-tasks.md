# Phase 3 - Chapter 2: Practice Tasks - HuggingFace Sentence-Transformers

## Task 1: Basic Embedding Generation Script

`all-MiniLM-L6-v2` model use karke ek Python script likho jo 5 sentences ka embedding generate kare aur unka shape print kare. Sentences aise choose karo jisme 2 semantically similar ho aur baaki alag topics pe ho.

## Task 2: Similarity Search Function

Ek function banao `find_most_similar(query, document_list)` jo:
- Query aur saare documents ko embed kare
- Cosine similarity calculate kare
- Top 3 most similar documents return kare (unke similarity scores ke saath, descending order me)

## Task 3: Model Comparison Experiment

`all-MiniLM-L6-v2` aur `all-mpnet-base-v2` dono models load karo, same 5 sentence pairs pe similarity score nikalo, aur compare karo ki dono models ke results kitne alag/same hain. Saath me note karo — dono models ko load/encode karne me kitna time (seconds) laga (`time` module use karke). Ye tumhe practically dikhayega ki resource trade-off real hai.

## Task 4: Apne QR System Me Apply Karo

Tumhare QR Food Ordering System me AI chatbot hai jo function calling aur Redis session memory use karta hai. Ab isme ek naya capability add karne ka socho:

- Apne restaurant ke menu items (naam + description) ki ek list banao (kam se kam 10 items, dummy data chalega)
- Sentence-Transformers se in sabke embeddings generate karo aur ek Python dictionary/list me store karo (abhi vector DB nahi use karna, wo Chapter 3-4 me aayega)
- Ek function likho `search_menu(user_query)` jo user ke natural language query (jaise "kuch spicy vegetarian dedo") ko embed kare aur menu items me se top 3 most relevant items return kare cosine similarity ke basis pe
- Socho aur likho (comment ke form me): ye semantic search tumhare current chatbot ke function-calling approach se kaise different hai, aur RAG pipeline me ye kahan fit hoga jab tum Chapter 3-6 complete kar loge