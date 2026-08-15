# Phase 1 → Chapter 1: How LLMs Work

## Kya/Kyu/Kaise

**Kya hai LLM?**
LLM (Large Language Model) ek aisa neural network hai jo bahut saare text pe train hua hai — internet, books, code, articles — aur uska ek hi kaam hai: **"is sequence ke baad agla word/token kya aayega, uski prediction karna."**

Bas itna simple hai core me. Baaki jo bhi magic dikhta hai (coding karna, sawal ka jawab dena, reasoning karna) — sab isi "next token predict karo" ke upar layered hai.

**Kyu samajhna zaroori hai?**
Agar tumhe ye pata nahi ki LLM andar se "next word guess karne wali machine" hai, toh tum prompt engineering, hallucination, context window jaisi cheezein sahi se debug nahi kar paoge. Interview me bhi ye poocha jata hai: *"LLM actually kaam kaise karta hai?"* — agar tum bol doge "AI jo sab jaanta hai" toh ye red flag hai.

**Kaise kaam karta hai (high level)?**
Tumhara input text tokens me tootta hai → model un tokens ko numbers (embeddings) me convert karta hai → layers of math (attention mechanism) chalti hai → output me agle token ki probability nikalti hai → highest probability wala (ya sampling se chuna) token generate hota hai → ye process repeat hota hai jab tak response complete na ho.

---

## 1. Tokens — LLM ka "word" nahi, "token" hota hai

LLM poore words nahi padhta — chhote pieces me todta hai jinhe **tokens** kehte hain. Ek token roughly ¾ word ke barabar hota hai (English me).

### Example:
```
Text: "Rajan is learning agentic AI"

Tokenized (approx):
["Raj", "an", " is", " learning", " agent", "ic", " AI"]

Total: 7 tokens for 5 words
```

Hindi/Hinglish text me tokens aur zyada lagte hain kyunki most LLMs primarily English-trained hain — Devanagari script ya mixed Hinglish inefficient tokenize hoti hai.

### Kyu matter karta hai:
- **Cost:** AI APIs (OpenAI, Anthropic) tokens ke hisaab se charge karte hain — input tokens + output tokens dono.
- **Context window limit:** Model ek baar me kitne tokens "yaad" rakh sakta hai, uski limit hoti hai.
- **Performance:** Zyada tokens = zyada processing time = zyada latency.

### Practical check:
Tum [OpenAI Tokenizer tool](https://platform.openai.com/tokenizer) pe koi bhi text paste karke dekh sakte ho ki wo kitne tokens me todta hai.

---

## 2. Context Window — LLM ki "short-term memory"

Context window = ek single conversation/request me LLM kitne tokens tak "dekh" sakta hai (input + output dono milakar).

### Example:
Agar ek model ka context window **128,000 tokens** hai (jaise GPT-4o), toh:
- Tumhara poora conversation history
- + System prompt
- + Current message
- + Model ka generate ho raha response

...sab milakar 128,000 tokens se zyada nahi ho sakta. Agar ho gaya, toh purana context "bhool" jata hai (truncate ho jata hai) ya error aata hai.

### Real-world analogy:
Socho tumhare paas ek **whiteboard** hai jispe sirf itni jagah hai ki 100 lines likh sako. Jab 101st line likhni ho, toh pehli line mitani padegi. LLM ka context window bhi aisa hi kaam karta hai — fixed size ka "working memory."

### Kyu ye tumhare QR System project se related hai:
Tumhare QR Food Ordering System me Redis session memory use hui hai — wahi is problem ko solve karti hai. Poori conversation history database/Redis me store hoti hai, aur har request pe **sirf relevant/recent part** LLM ko context window me bheja jata hai — taaki limit exceed na ho.

---

## 3. Temperature — Randomness ka control

Temperature ek number hota hai (usually 0 se 2 ke beech) jo decide karta hai ki LLM apna next-token prediction kitna "predictable" ya "creative" banaye.

### Kaise kaam karta hai:
Model har next token ke liye probability distribution generate karta hai. Jaise:

```
Prompt: "The sky is"

Model ke possible next tokens (probability):
"blue"  → 70%
"clear" → 15%
"dark"  → 10%
"pink"  → 5%
```

- **Temperature = 0**: Hamesha sabse high-probability token chunega ("blue" har baar). Deterministic, predictable output.
- **Temperature = 1 (default)**: Probabilities ke hisaab se randomly sample karta hai — kabhi "blue," kabhi "clear."
- **Temperature = 2 (high)**: Bahut random — kam-probability wale tokens bhi chun sakta hai ("pink"), output creative but sometimes nonsensical ho sakta hai.

### Kab kya use karna:
| Use Case | Temperature |
|---|---|
| Code generation, factual Q&A, structured data extraction | 0 - 0.3 (low, predictable chahiye) |
| General chatbot conversation | 0.7 (default, balanced) |
| Creative writing, brainstorming, story generation | 1.0 - 1.5 (high, variety chahiye) |

**Interview tip:** Agar poochein "production RAG system me temperature kya rakhoge?" — answer hai **low (0-0.3)**, kyunki tumhe factual, consistent answers chahiye, hallucination kam karni hai.

---

## 4. Next-Token Prediction — Poora process ek example se

Chalo ek poora cycle dekhte hain:

```
User input: "Capital of France is"

Step 1: Tokenize input
["Capital", " of", " France", " is"]

Step 2: Model process karta hai (attention layers, embeddings)
→ Har token ka context ke saath relationship samjha jata hai

Step 3: Next token probability nikalta hai
" Paris" → 85%
" a"     → 5%
" the"   → 3%
(others) → 7%

Step 4: Token select hota hai (temperature ke hisaab se)
→ " Paris" chuna gaya (high probability + low temp)

Step 5: Ye naya token input me add ho jata hai
"Capital of France is Paris"

Step 6: Process repeat — agla token predict hota hai
" Paris" ke baad shayad "." (full stop) predict hoga

→ Ye loop chalta rehta hai jab tak model "stop" signal na de
   (ya max tokens limit na aa jaaye)
```

Yehi reason hai LLM "generate" karta hai — ek-ek token, sequentially, har baar poora context dekhkar agla token guess karta hai.

---

## Common Misconceptions (Interview me pooche jate hain)

1. **"LLM ko sab kuch yaad rehta hai"** — Galat. Sirf current context window tak ki "memory" hoti hai, us se bahar kuch nahi.
2. **"LLM google search karta hai answer ke liye"** — Galat (bina tools ke). Base LLM sirf training data se seekhe patterns pe based predict karta hai, real-time internet access nahi hota jab tak tool/RAG na diya jaye.
3. **"Temperature 0 ka matlab LLM bilkul sahi jawab dega"** — Galat. Temperature sirf randomness control karta hai, factual correctness guarantee nahi karta. Isiliye hallucination temp=0 pe bhi ho sakta hai.

---

## Quick Recap (Apne alfaazon me likhna — interview prep)

- Token = ______________
- Context window = ______________
- Temperature high vs low ka trade-off = ______________
- Next-token prediction ka core loop = ______________

(Ye blanks khud fill karo apni understanding check karne ke liye — agar atak jao, upar wapas padho.)