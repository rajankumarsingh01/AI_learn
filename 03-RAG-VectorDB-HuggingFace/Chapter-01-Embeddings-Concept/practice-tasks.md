# Phase 3 → Chapter 1: Embeddings Concept — Practice Tasks

---

## Task 1: Conceptual Similarity Prediction (15 min)

Bina koi code chalaye, in text pairs ko dekho aur predict karo — high similarity hoga ya low (apna reasoning likho):

1. "Paneer tikka is delicious" vs "Paneer starter tastes great"
2. "Order status pending hai" vs "Mera order abhi tak process nahi hua"
3. "Best pizza in town" vs "Weather is nice today"
4. "Cancel my order" vs "I want to cancel"
5. "Cheap and affordable" vs "Expensive and premium"

**Deliverable:** Har pair ke liye apna prediction (High/Medium/Low similarity) aur ek line reasoning likho. Ye tumhara intuition build karega ki embeddings kaise "sochte" hain.

---

## Task 2: HuggingFace Embedding Model Se Real Test Karo (35 min)

Python environment setup karo (Phase 0 se pip/venv yaad karo):

```bash
pip install sentence-transformers --break-system-packages
```

```python
from sentence_transformers import SentenceTransformer
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np

# Chhota, lightweight model - hardware-friendly
model = SentenceTransformer('all-MiniLM-L6-v2')

texts = [
    "Paneer tikka is delicious",
    "Paneer starter tastes great",
    "Order status pending hai",
    "Mera order abhi tak process nahi hua",
    "Best pizza in town",
    "Weather is nice today"
]

embeddings = model.encode(texts)
print("Embedding shape:", embeddings.shape)  # (6, 384) - 384 dimensions

# Similarity calculate karo
similarity_matrix = cosine_similarity(embeddings)
print(similarity_matrix)
```

Run karo aur dekho — kya tumhare Task 1 wale predictions match karte hain actual similarity scores se?

**Deliverable:** Similarity matrix ka output paste karo, aur compare karo apne manual predictions ke saath — kaha match hua, kaha nahi.

---

## Task 3: Embedding Dimensions Explore Karo (15 min)

1. `embeddings.shape` print karke dekho kitne dimensions hain is model ke (`all-MiniLM-L6-v2`).
2. Ek single embedding print karo (`embeddings[0]`) — dekho actual numbers kaise dikhte hain.
3. Google karo `all-MiniLM-L6-v2` model ki details — kitna RAM/storage lagta hai, kya ye tumhare hardware (8GB RAM) ke liye suitable hai?

**Deliverable:** Model ke dimensions, aur apna assessment ki kya ye tumhare setup ke liye practical hai.

---

## Task 4: Apna Chhota Use-Case Socho (20 min)

Socho tumhe apne Sankalp/Kaksha app ke liye ek "FAQ search" feature banana hai — jaha student koi question type kare, aur system semantically closest FAQ dhundh ke de.

1. 5-6 sample FAQs likho (jaise "Fee payment kaise kare", "Class timing kya hai", etc.)
2. 3 different phrasings me user queries likho (jo exact FAQ text se match na ho, lekin meaning same ho)
3. Manually predict karo — kaunsi FAQ kaunse query se sabse zyada match karegi

**Deliverable:** Apna FAQ list + queries + predictions likho. Agar time ho, Task 2 wale code me inhe daal ke actual similarity nikaal ke verify karo apne predictions.

---

**Note:** Agla chapter (HuggingFace Sentence-Transformers) isi model/library ko deeply explore karega — production-ready embedding pipeline banane ke liye.