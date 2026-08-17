# Phase 3 - Chapter 6: Full RAG Pipeline Assembly (End-to-End)

## Kya Hai Ye?

Ye Phase 3 ka final chapter hai — ab tak humne RAG (Retrieval-Augmented Generation) ke individual pieces seekhe: Embeddings (Ch1), Sentence-Transformers (Ch2), ChromaDB (Ch3), FAISS (Ch4), Chunking (Ch5). Is chapter me hum in sabko jodkar ek complete, end-to-end RAG pipeline banayenge — jo real document leke, usse chunk kare, embed kare, vector DB me store kare, aur query aane pe relevant context nikaal ke LLM ko de, taaki wo accurate, grounded answer generate kare.

RAG ka poora point ye hai — LLM apna answer sirf apni training data se nahi, balki tumhare diye gaye **specific, up-to-date documents** se generate kare. Isse hallucination kam hota hai aur answers factually grounded hote hain.

## Kyu Zaroori Hai?

1. **Interview Ka Sabse Common Practical Question**: "RAG pipeline design karo" ya "explain end-to-end RAG flow" — ye almost har AI/ML-adjacent interview me pucha jaata hai. Isse practically implement kar chuke hone ka matlab hai tum confidently answer de sakoge.

2. **Portfolio Project Ready Karna**: Tumhare teeno projects (QR System, Kaksha, API Observability) me se kisi me bhi RAG add karna ek strong differentiator hoga — jaise QR System me "ask about menu/policies" chatbot feature, ya Kaksha me "ask about course content" feature.

3. **LangChain Se Pehle Fundamentals Samajhna**: Phase 5 me tum LangChain seekhoge, jo ye sara kaam abstract kar deta hai (bilt-in chunking, retrieval chains). Lekin agar tumne pehle isse manually banaya hai, to LangChain "andar se kya kar raha hai" wo clearly samjhoge — jo interview me deep-dive questions ke liye zaroori hai.

4. **Real-World System Design Skill**: RAG pipeline banana ek chhota system design exercise hai — data flow, error handling, performance considerations sab involve hote hain. Ye skill directly transferable hai kisi bhi production AI feature banane me.

## Kaise Kaam Karta Hai?

RAG pipeline do phases me divide hota hai:

**Indexing Phase (ek baar, ya jab data update ho)**: Document → Chunk (Ch5) → Embed (Ch2) → Vector DB me Store (Ch3)

**Query Phase (har user request pe)**: User Query → Embed → Vector DB se Retrieve (top-k relevant chunks) → LLM Prompt me Context inject karo → LLM Response generate kare

---

## 1. Complete Pipeline - Indexing Phase

```python
import chromadb
from sentence_transformers import SentenceTransformer

# Setup
model = SentenceTransformer('all-MiniLM-L6-v2')
client = chromadb.PersistentClient(path="./rag_db")
collection = client.get_or_create_collection(
    name="knowledge_base",
    metadata={"hnsw:space": "cosine"}
)

def chunk_with_metadata(text, source_doc, chunk_size=400, overlap=50):
    chunks = []
    start = 0
    idx = 0
    while start < len(text):
        end = start + chunk_size
        chunk_text = text[start:end]
        chunks.append({
            "text": chunk_text,
            "id": f"{source_doc}_chunk_{idx}",
            "metadata": {"source": source_doc, "chunk_index": idx}
        })
        start = end - overlap
        idx += 1
    return chunks

def index_document(text, source_name):
    chunks = chunk_with_metadata(text, source_name)
    
    texts = [c["text"] for c in chunks]
    ids = [c["id"] for c in chunks]
    metadatas = [c["metadata"] for c in chunks]
    
    embeddings = model.encode(texts).tolist()
    
    collection.add(
        embeddings=embeddings,
        documents=texts,
        ids=ids,
        metadatas=metadatas
    )
    print(f"Indexed {len(chunks)} chunks from {source_name}")

# Usage
document_text = "..."  # tumhara lamba document
index_document(document_text, source_name="qr_system_policies")
```

## 2. Complete Pipeline - Retrieval Phase

```python
def retrieve_context(query, top_k=3, filter_metadata=None):
    query_embedding = model.encode([query]).tolist()
    
    results = collection.query(
        query_embeddings=query_embedding,
        n_results=top_k,
        where=filter_metadata
    )
    
    retrieved_chunks = results['documents'][0]
    sources = [m['source'] for m in results['metadatas'][0]]
    
    return retrieved_chunks, sources

# Usage
query = "refund kaise milega agar order cancel karna ho"
chunks, sources = retrieve_context(query, top_k=3)
```

## 3. Complete Pipeline - Generation Phase (LLM Ko Context Dena)

```python
def build_prompt(query, retrieved_chunks):
    context = "\n\n".join(retrieved_chunks)
    
    prompt = f"""Neeche diye gaye context ka use karke user ke question ka answer do.
Agar context me answer nahi hai, to saaf bata do ki information available nahi hai.

Context:
{context}

Question: {query}

Answer:"""
    return prompt

# Ye prompt phir LLM API (jaise Anthropic/OpenAI, Chapter Phase 1 yaad karo) ko bheja jaata hai
final_prompt = build_prompt(query, chunks)
```

Yahan Phase 1 Chapter 2 (OpenAI/Anthropic API) ka knowledge use hoga — is `final_prompt` ko LLM API call me bhejna, jaisa tumne Phase 1 me seekha tha.

## 4. Full End-to-End Function

```python
def rag_query(user_query, top_k=3):
    # Step 1: Retrieve
    chunks, sources = retrieve_context(user_query, top_k=top_k)
    
    # Step 2: Build prompt
    prompt = build_prompt(user_query, chunks)
    
    # Step 3: Call LLM (pseudo-code, Phase 1 ka API call yahan aayega)
    # response = call_llm_api(prompt)
    
    return {
        "query": user_query,
        "retrieved_chunks": chunks,
        "sources": sources,
        "prompt_sent_to_llm": prompt
        # "answer": response
    }

result = rag_query("delivery me kitna time lagta hai")
print(result)
```

## 5. RAG Pipeline Ke Common Failure Points (Debugging Ke Liye)

| Problem | Kaha Se Aata Hai | Fix |
|---|---|---|
| Irrelevant chunks retrieve ho rahe hain | Chunk size galat, ya embedding model weak | Ch5 chunking revisit karo, ya better model try karo |
| "Context me nahi mila" baar-baar | `top_k` bahut kam hai, ya document indexed hi nahi hua | `top_k` badhao, verify karo `collection.count()` |
| Slow retrieval | Bahut zyada documents, Flat index use ho raha | Ch4 FAISS IVF index consider karo, ya ChromaDB HNSW tuning |
| LLM hallucinate kar raha, context ignore kar raha | Prompt me instruction weak hai | Prompt me explicitly bolo "sirf context se answer do" |
| Same query pe different results | Non-deterministic embedding, ya collection corrupt | Model version pin karo, collection re-verify karo |

## 6. Evaluation - Kaise Pata Chalega RAG Accha Kaam Kar Raha Hai?

Basic manual evaluation approach (Phase 7 me "Evals" detail se aayega, yahan intro):

```python
test_cases = [
    {"query": "refund policy kya hai", "expected_source": "qr_system_policies"},
    {"query": "delivery time kitna hai", "expected_source": "qr_system_policies"},
]

def evaluate_retrieval(test_cases):
    correct = 0
    for case in test_cases:
        _, sources = retrieve_context(case["query"], top_k=1)
        if case["expected_source"] in sources:
            correct += 1
    accuracy = correct / len(test_cases)
    print(f"Retrieval Accuracy: {accuracy * 100}%")
```

Ye simple retrieval accuracy check hai — production systems me isse zyada sophisticated metrics (precision, recall, relevance scoring) use hote hain, jo Phase 7 me cover hoga.

## Common Mistakes (Interview me pooche jate hain)

1. **"Retrieval aur Generation ko alag steps na samajhna"** — RAG do distinct phases hai (retrieve, phir generate). Beginners inhe ek hi step samajh lete hain, jabki har phase independently debug/optimize ho sakta hai — jaise retrieval sahi hai lekin LLM prompt weak hai, ya retrieval hi galat chunks laa raha hai.

2. **"Prompt me explicit instruction na dena ki sirf context use karo"** — Agar LLM ko clearly nahi bataya "sirf diye gaye context se answer do", to wo apni training data se bhi mix kar sakta hai — jisse hallucination ka risk badhta hai jo RAG ka poora purpose defeat karta hai.

3. **"Indexing aur Query phase ke embeddings alag model se generate karna"** — Ye Chapter 2-4 me discuss kiya tha, lekin full pipeline me ye mistake repeat hoti hai jab log query ke liye kabhi galti se doosra model call kar dete hain.

4. **"top_k ko fixed rakhna sabhi queries ke liye"** — Kuch queries broad hoti hain (zyada context chahiye), kuch specific (kam context kaafi hai). Advanced RAG systems dynamically `top_k` adjust karte hain ya re-ranking step add karte hain.

5. **"Evaluation skip kar dena"** — RAG pipeline bana lene ke baad ye assume kar lena ki "kaam kar raha hai" bina systematically test kiye — production me ye risky hai. Chhote test cases ke saath retrieval accuracy check karna basic hygiene hai.