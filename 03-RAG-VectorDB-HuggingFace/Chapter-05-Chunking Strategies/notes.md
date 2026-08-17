# Phase 3 - Chapter 5: Chunking Strategies

## Kya Hai Ye?

Chunking ka matlab hai ek bade document (jaise ek PDF, article, ya knowledge base page) ko chhote-chhote pieces ("chunks") me todna, taaki har chunk ko separately embed karke vector DB me store kiya ja sake. Abhi tak humne (Chapter 2-4 me) chhote sentences/documents pe kaam kiya — real world me documents lambe hote hain (jaise ek poora FAQ page, ya SentinelAI ka security policy document), aur unhe as-it-is embed karna practical nahi hota.

Chunking strategy ka direct impact RAG system ki quality pe padta hai — galat chunking se ya to context incomplete milta hai (chunk bahut chhota) ya irrelevant info mix ho jaati hai (chunk bahut bada).

## Kyu Zaroori Hai?

1. **Embedding Model Ki Token Limit**: Sentence-Transformers models (jaise `all-MiniLM-L6-v2`) ki ek maximum input length hoti hai (typically 256-512 tokens). Isse zyada lamba text truncate ho jaata hai — matlab document ka baaki hissa embedding me capture hi nahi hota.

2. **Retrieval Precision**: Agar poora document ek hi chunk hai, to query se match hone pe pura document LLM ko context me jaayega — jisme sirf ek chhota relevant part hoga aur baaki irrelevant noise. Chhote, focused chunks se precise retrieval hota hai.

3. **Context Window Efficiency**: LLM (jaise Claude/GPT) ko context bhejte time token limit aur cost dono matter karte hain. Sahi-size chunks se sirf relevant information context me jaati hai, na ki pura bada document.

4. **Semantic Coherence**: Agar chunking galat jagah se todi jaaye (jaise ek sentence beech me kat jaaye), to embedding ka meaning distort ho jaata hai — retrieval quality kharab hoti hai.

5. **Tumhare Projects Ke Liye Directly Relevant**: SentinelAI (security docs), Kaksha (course content/study material), QR System (menu descriptions, policies) — sabme agar tum lambe documents ko RAG me daalna chaho, chunking strategy decide karegi ki retrieval kitna accurate hoga.

## Kaise Kaam Karta Hai?

Chunking algorithm document ko text units (characters, words, sentences, paragraphs) ke basis pe todta hai, kabhi-kabhi thoda **overlap** rakhte hue taaki context continuity na tootey chunks ke beech.

---

## 1. Fixed-Size Chunking (Simplest Approach)

```python
def fixed_size_chunk(text, chunk_size=500, overlap=50):
    chunks = []
    start = 0
    while start < len(text):
        end = start + chunk_size
        chunks.append(text[start:end])
        start = end - overlap  # overlap se continuity maintain hoti hai
    return chunks

document = "..." # tumhara lamba document text
chunks = fixed_size_chunk(document, chunk_size=500, overlap=50)
print(f"Total chunks: {len(chunks)}")
```

**Problem**: Ye character count pe blindly todta hai — kisi sentence ke beech me bhi chunk boundary aa sakti hai, jo semantic meaning distort karta hai.

## 2. Sentence-Based Chunking (Better Semantic Boundaries)

```python
import re

def sentence_chunk(text, max_sentences=5):
    sentences = re.split(r'(?<=[.!?])\s+', text)
    chunks = []
    for i in range(0, len(sentences), max_sentences):
        chunk = ' '.join(sentences[i:i + max_sentences])
        chunks.append(chunk)
    return chunks
```

Ye approach sentence boundaries respect karta hai — koi sentence beech me nahi katega. Lekin agar sentences bahut lambe/chhote hon, to chunk sizes inconsistent ho sakte hain.

## 3. Recursive Character Splitting (Industry Standard Approach)

Ye approach LangChain jaise frameworks me widely use hoti hai (jo tum Phase 5 me detail se seekhoge) — idea ye hai ki pehle bade separators try karo (paragraph breaks), agar chunk phir bhi bada hai to chhote separators pe fallback karo (sentence, phir word):

```python
def recursive_split(text, chunk_size=500, separators=["\n\n", "\n", ". ", " "]):
    if len(text) <= chunk_size:
        return [text]
    
    for sep in separators:
        if sep in text:
            parts = text.split(sep)
            chunks = []
            current_chunk = ""
            for part in parts:
                if len(current_chunk) + len(part) <= chunk_size:
                    current_chunk += part + sep
                else:
                    if current_chunk:
                        chunks.append(current_chunk.strip())
                    current_chunk = part + sep
            if current_chunk:
                chunks.append(current_chunk.strip())
            return chunks
    
    # Agar koi separator nahi mila, force split karo
    return [text[i:i+chunk_size] for i in range(0, len(text), chunk_size)]
```

Ye approach paragraph → line → sentence → word ka hierarchy follow karta hai, jisse chunks naturally meaningful boundaries pe bante hain jab tak possible ho.

## 4. Overlap Ka Role — Kyu Zaroori Hai

```python
# Bina overlap ke
chunk1 = "...aur is API ka rate limit 100 requests per minute hai."
chunk2 = "Isse zyada requests aane par 429 error return hota hai."
# Problem: chunk2 akela padhne pe pata nahi chalega "isse" kis cheez ko refer kar raha hai

# Overlap ke saath (50 characters overlap)
chunk1 = "...aur is API ka rate limit 100 requests per minute hai."
chunk2 = "...rate limit 100 requests per minute hai. Isse zyada requests aane par 429 error return hota hai."
# Ab context clear hai, kyunki previous sentence bhi included hai
```

Typical overlap 10-20% of chunk size hota hai. Zyada overlap = better context continuity, lekin storage/compute cost bhi badhta hai (duplicate content).

## 5. Chunk Size Choose Karna — Trade-offs

| Chunk Size | Pros | Cons |
|---|---|---|
| Chhota (100-200 tokens) | Precise retrieval, focused context | Context incomplete ho sakta hai, zyada chunks = zyada storage |
| Medium (300-500 tokens) | Balance — most RAG use cases ke liye default | - |
| Bada (800+ tokens) | Zyada context per chunk, kam chunks | Irrelevant info mix ho sakti hai, embedding quality degrade ho sakti hai (average out ho jaata hai meaning) |

**Rajan ke liye recommendation**: 300-500 characters/tokens se start karo with 10-15% overlap — ye most practical RAG applications ke liye sweet spot hai. Phase 3 Chapter 6 (full pipeline) me tum ise apne actual use case ke hisaab se tune karoge.

## 6. Metadata Ke Saath Chunk Karna (Best Practice)

Sirf text chunk karna kaafi nahi — original document ka reference bhi rakhna chahiye:

```python
def chunk_with_metadata(text, source_doc, chunk_size=500, overlap=50):
    raw_chunks = fixed_size_chunk(text, chunk_size, overlap)
    result = []
    for i, chunk in enumerate(raw_chunks):
        result.append({
            "text": chunk,
            "metadata": {
                "source": source_doc,
                "chunk_index": i,
                "total_chunks": len(raw_chunks)
            }
        })
    return result
```

Ye metadata ChromaDB me store karte time useful hota hai (Chapter 3 yaad karo) — retrieval ke baad user ko batana ki information kaunse document/section se aayi.

## Common Mistakes (Interview me pooche jate hain)

1. **"Overlap bilkul use na karna"** — Bina overlap ke, chunk boundaries pe context kat jaata hai, aur retrieval quality kharab hoti hai kyunki adjacent chunks ek doosre se disconnected ho jaate hain.

2. **"Fixed-size chunking ko hamesha best samajh lena"** — Character-count based chunking simple hai lekin semantically naive hai — sentences/paragraphs beech me kat sakte hain. Production RAG systems generally recursive ya semantic chunking use karte hain.

3. **"Chunk size ko document type ke hisaab se adjust na karna"** — Code documentation, legal documents, aur chat conversations sabki natural structure alag hoti hai — same fixed chunk_size sabke liye optimal nahi hoga. Interview me ye demonstrate karna important hai ki tum content-aware decisions le sakte ho.

4. **"Metadata track na karna"** — Sirf raw text chunk karke store karna kaafi nahi, source document, chunk position, aur original context ka reference rakhna zaroori hai — warna retrieval ke baad "ye information kahan se aayi" trace karna mushkil ho jaata hai.

5. **"Embedding model ki max token limit ignore karna"** — Agar chunk size embedding model ke max token limit se bada hai, to model text ko silently truncate kar dega — matlab chunk ka baaki hissa embedding me represent hi nahi hoga, bina kisi error/warning ke. Ye silent bug interview me discuss karna impressive lagta hai.