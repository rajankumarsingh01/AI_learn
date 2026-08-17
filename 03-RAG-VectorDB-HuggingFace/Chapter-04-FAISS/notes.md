# Phase 3 - Chapter 4: FAISS (Facebook AI Similarity Search)

## Kya Hai Ye?

FAISS (Facebook AI Similarity Search) ek library hai jo Meta (Facebook) ne banayi hai specifically **large-scale vector similarity search** ke liye. Chapter 3 me humne ChromaDB seekha jo ek full-fledged vector database hai (storage + indexing + metadata + persistence sab kuch built-in). FAISS ismein different hai — ye sirf ek **indexing/search library** hai, database nahi. Iska matlab: FAISS bahut fast aur memory-efficient hai similarity search ke liye, lekin persistence, metadata filtering jaisi cheezein tumhe khud handle karni padti hain.

Socho aise — ChromaDB ek pura restaurant hai (kitchen + waiter + billing sab included), FAISS sirf ek super-fast chef hai (bas cooking, baaki sab tumhe manage karna hai).

## Kyu Zaroori Hai?

1. **Raw Speed & Scale**: FAISS lakhon-crodon vectors pe bhi milliseconds me search kar sakta hai — ye industry-standard hai large-scale similarity search ke liye (Facebook khud isse billions of vectors pe use karta hai).

2. **Multiple Index Types**: FAISS alag-alag indexing algorithms deta hai jo speed vs accuracy trade-off ko fine-tune karne dete hain — chhote dataset ke liye exact search, bade dataset ke liye approximate search.

3. **Interview Relevance**: Bahut saari companies (especially jo apna custom RAG/search infra banate hain) FAISS directly use karti hain kyunki ye ChromaDB se zyada control aur performance deta hai. Interview me FAISS ka concept aana common hai, especially "how would you scale vector search" jaise questions me.

4. **Lightweight & Local**: FAISS bhi CPU pe achhe se chalta hai (GPU optional), isliye tumhare i3/8GB setup ke liye suitable hai — bas dataset size ka dhyan rakhna padega.

5. **Understanding Under the Hood**: ChromaDB internally bhi HNSW jaisa concept use karta hai jo FAISS ke concepts se related hai. FAISS seekhne se tumhe samajh aayega ki vector databases "andar se" kaise kaam karte hain — jo interview me deep technical questions ke liye zaroori hai.

## Kaise Kaam Karta Hai?

FAISS vectors ko ek "index" me organize karta hai — jo essentially ek data structure hai jo similarity search ko fast banata hai. Sabse simple index hai **Flat Index** (brute-force, exact search), aur advanced indexes (IVF, HNSW) approximate search karte hain speed ke liye thoda accuracy trade-off karke.

---

## 1. Installation

```bash
pip install faiss-cpu
```

`faiss-cpu` use karo (GPU version `faiss-gpu` tumhare i3 system ke liye zaroori nahi, aur GPU bhi nahi hai).

## 2. Basic Flat Index (Exact Search)

```python
import faiss
import numpy as np
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')

documents = [
    "Paneer Tikka - spicy grilled cottage cheese",
    "Veg Biryani - fragrant rice with vegetables",
    "Cold Coffee - chilled coffee with ice cream",
    "Butter Naan - soft bread with butter"
]

embeddings = model.encode(documents)
embeddings = np.array(embeddings).astype('float32')  # FAISS ko float32 chahiye

dimension = embeddings.shape[1]  # 384 for MiniLM

# Flat index - exact brute force search, L2 (Euclidean) distance
index = faiss.IndexFlatL2(dimension)
index.add(embeddings)

print(index.ntotal)  # Total vectors stored: 4
```

## 3. Query Karna

```python
query = "kuch spicy chahiye"
query_embedding = model.encode([query]).astype('float32')

k = 2  # top-2 results chahiye
distances, indices = index.search(query_embedding, k)

print(indices)    # [[0, 1]] -> documents[0] aur documents[1] ke indexes
print(distances)  # [[0.45, 0.89]] -> unki L2 distances

for idx in indices[0]:
    print(documents[idx])
```

Notice karo — FAISS sirf **index numbers** return karta hai, actual text nahi. Tumhe khud ek separate list/dictionary maintain karni padti hai jo index → original document map kare. Ye ChromaDB se bada farak hai, jahan `documents` aur `metadata` automatically saath aate hain.

## 4. Cosine Similarity Ke Liye Index (Normalized Vectors)

FAISS by default L2 (Euclidean) distance use karta hai. Cosine similarity ke liye, embeddings ko normalize karna padta hai aur `IndexFlatIP` (Inner Product) use karna padta hai:

```python
import faiss

# Embeddings normalize karo (unit length banao)
faiss.normalize_L2(embeddings)

# Inner Product index (normalized vectors ke saath ye cosine similarity ke barabar hai)
index = faiss.IndexFlatIP(dimension)
index.add(embeddings)

# Query bhi normalize karna mat bhoolna
query_embedding = model.encode([query]).astype('float32')
faiss.normalize_L2(query_embedding)

scores, indices = index.search(query_embedding, k)
# Yahan scores HIGHER hone chahiye better match ke liye (L2 distance ke ulta)
```

Ye ek common interview trap hai — Chapter 3 me humne dekha tha distance me lower = better, lekin yahan Inner Product me higher = better. Confusion na ho isliye dhyan rakho.

## 5. IVF Index - Large Scale Ke Liye (Approximate Search)

Jab dataset bahut bada ho (lakhon vectors), Flat Index slow ho jaata hai kyunki ye brute-force compare karta hai. **IVF (Inverted File Index)** data ko clusters me divide karke search ko fast banata hai:

```python
nlist = 10  # kitne clusters banane hain (dataset size ke hisaab se choose karo)
quantizer = faiss.IndexFlatL2(dimension)
index = faiss.IndexIVFFlat(quantizer, dimension, nlist)

# IVF index ko train karna padta hai (clusters seekhne ke liye)
index.train(embeddings)
index.add(embeddings)

index.nprobe = 3  # kitne clusters search karte time check karne hain (speed vs accuracy trade-off)

distances, indices = index.search(query_embedding, k)
```

**Rajan ke liye reality check**: Tumhare current projects (QR System jaise) me document count chhota hai (dus-sau items), isliye `IndexFlatL2`/`IndexFlatIP` hi kaafi hai — IVF tab relevant hoga jab tumhare paas lakhon documents ho. Lekin concept samajhna interview ke liye zaroori hai.

## 6. Index Save & Load (Persistence)

FAISS khud persistence handle nahi karta jaise ChromaDB karta hai, lekin manually save/load kar sakte ho:

```python
# Save
faiss.write_index(index, "menu_index.faiss")

# Load
loaded_index = faiss.read_index("menu_index.faiss")
```

Documents/metadata alag se (jaise ek JSON file ya pickle me) save karne padenge, kyunki FAISS sirf vectors store karta hai, text nahi.

## 7. FAISS vs ChromaDB - Kab Kya Use Karo?

| Feature | ChromaDB | FAISS |
|---|---|---|
| Setup complexity | Easy (batteries included) | Thoda manual (documents/metadata khud manage karo) |
| Persistence | Built-in | Manual (`write_index`/`read_index`) |
| Metadata filtering | Built-in (`where` clause) | Nahi hai — khud implement karna padega |
| Speed at massive scale | Achha | Best-in-class (industry standard) |
| Best for | Rapid prototyping, small-medium RAG apps | Custom high-performance search systems |

**Rajan ke liye recommendation**: Apne portfolio projects (QR System, Kaksha) me ChromaDB use karo — kam setup, zyada features. FAISS ka concept interview ke liye seekho aur samajh ke rakho ki "jab scale bahut bada ho aur fine control chahiye ho, to FAISS jaisa raw library approach better hai."

## Common Mistakes (Interview me pooche jate hain)

1. **"FAISS ko complete database samajh lena"** — FAISS sirf indexing/search library hai, DB nahi. Persistence, metadata, CRUD operations sab khud implement karne padte hain. Interview me isse ChromaDB/Pinecone jaise "vector databases" se differentiate karna zaroori hai.

2. **"Embeddings ko float32 me convert na karna"** — FAISS ko strictly `float32` numpy arrays chahiye. Agar `float64` ya list pass kar doge, error aayega ya unexpected behavior hoga.

3. **"Normalize karna bhool jaana Inner Product index ke saath"** — `IndexFlatIP` sirf tab cosine similarity ke equivalent hota hai jab vectors normalized (unit length) hon. Bina normalize kiye, ye sirf raw dot product hoga jo magnitude se bhi affect hota hai — misleading results denge.

4. **"IVF index ko train() kiye bina use karna"** — `IndexIVFFlat` jaise approximate indexes ko `add()` karne se pehle `train()` karna zaroori hai (taaki clusters seekh sake). Ye step miss karne pe error aayega ya galat results milenge.

5. **"nprobe ka effect na samajhna"** — `nprobe` value jitni zyada, utni accuracy better lekin speed slow. Interview me pooch sakte hain "IVF index me speed-accuracy trade-off kaise control karte ho" — answer hai `nprobe` parameter.