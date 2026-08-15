# Phase 3 - Chapter 3: Vector Databases - ChromaDB Basics

## Kya Hai Ye?

Vector Database ek special type ka database hai jo embeddings (vectors) ko efficiently store, index aur search karne ke liye design kiya gaya hai. Normal database (MongoDB, MySQL) me tum exact match ya range query karte ho ("naam = 'Rajan'"), lekin embeddings ke saath tumhe "similarity search" chahiye hoti hai — "kaunse vectors is query vector ke sabse close hain?"

Chapter 2 me humne Sentence-Transformers se embeddings generate karna seekha. Ab problem ye hai — agar tumhare paas 10,000 documents hain, to har query pe saare 10,000 embeddings ke saath manually cosine similarity calculate karna (jaise humne practice task me kiya) bahut slow ho jaayega. Vector DB ye problem solve karta hai — smart indexing (jaise HNSW algorithm) use karke milliseconds me relevant results deta hai, chahe lakhon documents ho.

**ChromaDB** ek open-source, lightweight vector database hai jo local machine pe (bina server setup ke) chal sakta hai — isliye tumhare i3/8GB Windows system ke liye perfect starting point hai.

## Kyu Zaroori Hai?

1. **Scalability**: Manual cosine similarity loop O(n) hai — jitne document badhenge, utna slow hoga. Vector DB approximate nearest neighbor (ANN) algorithms use karta hai jo bahut fast hai bade datasets pe bhi.

2. **Persistence**: Sentence-Transformers embeddings generate karta hai lekin unhe store nahi karta. Agar tum apni Python script restart karo, saare embeddings dobara generate karne padenge. ChromaDB embeddings ko disk pe persist karta hai.

3. **Metadata Filtering**: Sirf similarity search hi nahi, ChromaDB tumhe metadata ke saath filter bhi karne deta hai — jaise "sirf vegetarian items me se search karo" ya "sirf last 30 din ke documents me dhoondo".

4. **RAG Pipeline Ka Backbone**: Har production RAG system ka core hota hai — Document → Chunk → Embed → **Vector DB me Store** → Query → **Vector DB se Retrieve** → LLM ko context do. Ye chapter tumhare RAG pipeline ka missing piece complete karega.

5. **Tumhare Hardware Ke Liye Ideal**: ChromaDB embedded mode me chalta hai (SQLite jaisa) — koi separate server process nahi chahiye, RAM footprint bhi kam hai. Pinecone/Weaviate jaise cloud vector DBs abhi ke liye avoid karo, ChromaDB se hi seekhna shuru karo.

## Kaise Kaam Karta Hai?

ChromaDB internally embeddings ko ek "collection" me store karta hai (SQL table jaisa concept), aur query time pe HNSW (Hierarchical Navigable Small World) indexing algorithm use karke approximate nearest neighbors dhoondta hai — bina saare vectors ko brute-force check kiye.

---

## 1. Installation & Setup

```bash
pip install chromadb
```

Ye lightweight install hai (Sentence-Transformers jitna heavy nahi), aur koi separate database server install karne ki zaroorat nahi.

## 2. Basic Client Setup - Persistent Storage

```python
import chromadb

# Persistent client - data disk pe save hoga
client = chromadb.PersistentClient(path="./chroma_db")

# Collection banao (table jaisa concept)
collection = client.get_or_create_collection(name="qr_menu_items")
```

`PersistentClient` use karna important hai — agar `chromadb.Client()` (in-memory) use karoge, to script band hote hi data gayab ho jayega. Production/practice dono ke liye `PersistentClient` hi use karo.

## 3. Documents Add Karna (with Auto-Embedding)

ChromaDB ka ek badiya feature hai — ye khud embedding generate kar sakta hai (default embedding function use karke), ya tum apna Sentence-Transformers embedding pass kar sakte ho:

```python
# Option A: ChromaDB khud embed karega (default model use karke)
collection.add(
    documents=[
        "Paneer Tikka - spicy grilled cottage cheese with mint chutney",
        "Veg Biryani - fragrant rice with mixed vegetables and spices",
        "Cold Coffee - chilled coffee with ice cream"
    ],
    ids=["item1", "item2", "item3"],
    metadatas=[
        {"category": "starter", "veg": True, "spicy": True},
        {"category": "main", "veg": True, "spicy": True},
        {"category": "beverage", "veg": True, "spicy": False}
    ]
)
```

```python
# Option B: Apna Sentence-Transformers embedding pass karo (zyada control)
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')
texts = ["Paneer Tikka - spicy grilled cottage cheese", "Veg Biryani - fragrant rice"]
embeddings = model.encode(texts).tolist()

collection.add(
    embeddings=embeddings,
    documents=texts,
    ids=["item1", "item2"]
)
```

**Rajan ke liye recommendation**: Option B use karo — kyunki tum already Chapter 2 me `all-MiniLM-L6-v2` seekh chuke ho, aur consistent embedding model use karna (jo tumne query ke liye bhi use karna hai) critical hai — Chapter 2 ke "dimension mismatch" mistake yaad hai na?

## 4. Query Karna (Similarity Search)

```python
query_text = "kuch spicy vegetarian dedo"
query_embedding = model.encode([query_text]).tolist()

results = collection.query(
    query_embeddings=query_embedding,
    n_results=3
)

print(results['documents'])   # Top 3 matching documents
print(results['distances'])   # Unki distances (kam distance = zyada similar)
print(results['metadatas'])   # Unka metadata
```

## 5. Metadata Filtering (`where` clause)

Ye feature manual cosine similarity approach me possible nahi tha — ye ChromaDB ka real power hai:

```python
results = collection.query(
    query_embeddings=query_embedding,
    n_results=3,
    where={"veg": True, "spicy": True}  # Sirf veg aur spicy items me search karo
)
```

Isse tum semantic search aur structured filtering dono ek saath kar sakte ho — jaise "spicy vegetarian items dhoondo jo query se match karte ho".

## 6. Update aur Delete Operations

```python
# Update
collection.update(
    ids=["item1"],
    documents=["Paneer Tikka - now less spicy, mild flavor"],
    metadatas=[{"category": "starter", "veg": True, "spicy": False}]
)

# Delete
collection.delete(ids=["item3"])

# Collection ke total items count karo
print(collection.count())
```

## 7. Distance Metrics: Cosine vs Euclidean vs Dot Product

ChromaDB collection banate time distance metric specify kar sakte ho:

```python
collection = client.get_or_create_collection(
    name="qr_menu_items",
    metadata={"hnsw:space": "cosine"}  # default hai "l2" (euclidean)
)
```

Chapter 2 me humne discuss kiya tha ki normalized embeddings ke liye cosine similarity best hoti hai — isliye RAG/semantic search use cases me `cosine` explicitly set karna best practice hai.

## Common Mistakes (Interview me pooche jate hain)

1. **"In-memory client production me use karna"** — `chromadb.Client()` (without path) sirf testing ke liye hai. Production/persistent use case me hamesha `PersistentClient(path=...)` use karo, warna restart pe data loss ho jayega.

2. **"Embedding model consistency na rakhna"** — Agar `collection.add()` ke time ek model use kiya aur query ke time doosra, to distances meaningless honge (dimension mismatch ya semantic space mismatch). Same model dono jagah use karna mandatory hai.

3. **"n_results ko bahut zyada rakhna"** — Beginners kabhi-kabhi `n_results=100` jaisa bada number rakh dete hain "safe side" ke liye, lekin ye LLM ko unnecessary context deta hai (token waste + confusion). RAG me typically `n_results=3-5` best practice hai, phir re-ranking se refine karo.

4. **"Distance ko similarity samajh lena"** — `results['distances']` LOWER value = MORE similar (agar l2/euclidean use kar rahe ho). Beginners isko ulta samajh lete hain aur sabse high distance wale result ko "best match" bol dete hain — interview me ye common trap hai.

5. **"Metadata filtering ka use na jaanna"** — Sirf embedding similarity pe depend karna kaafi nahi hota real applications me. `where` clause ka use na karna ek missed opportunity hai jo interview me demonstrate karna chahiye — ye shows ki tum sirf theoretical nahi, practical RAG design samajhte ho.