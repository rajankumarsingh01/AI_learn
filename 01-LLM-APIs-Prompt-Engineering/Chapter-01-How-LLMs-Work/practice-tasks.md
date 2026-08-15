# Phase 1 → Chapter 1: How LLMs Work — Practice Tasks

---

## Task 1: Tokenizer Exploration (15 min)

1. [OpenAI Tokenizer](https://platform.openai.com/tokenizer) tool kholo.
2. In teeno text ko paste karke token count note karo:
   - Pure English: `"I am a full stack developer learning agentic AI."`
   - Same meaning Hinglish me: `"Main ek full stack developer hoon jo agentic AI seekh raha hoon."`
   - Pure Hindi (Devanagari): `"मैं एक फुल स्टैक डेवलपर हूं जो एजेंटिक AI सीख रहा हूं।"`
3. Teeno ke token counts likho aur observe karo — Hindi/Hinglish English se kitna zyada tokens leta hai?

**Deliverable:** `notes.md` ke end me ek table bana ke apna result likho (English tokens vs Hinglish tokens vs Hindi tokens).

---

## Task 2: Temperature Simulation — Manual (20 min)

Bina API call kiye, sirf concept practice karne ke liye:

Neeche diye gaye probability distribution ko dekho:
```
Prompt: "My favorite programming language is"

Token probabilities:
"JavaScript" → 45%
"Python"     → 30%
"TypeScript" → 15%
"Rust"       → 10%
```

Likho (apne alfaazon me):
1. Temperature = 0 pe kaunsa token select hoga aur kyu?
2. Temperature = 1.5 pe kya ho sakta hai — kya "Rust" bhi chuna ja sakta hai? Reasoning likho.
3. Agar ye ek **production customer-support chatbot** hai jo hamesha company policy ke exact wording follow karna chahiye, toh kaunsa temperature use karoge? One line justification.

---

## Task 3: Context Window Math (15 min)

Maan lo ek model ka context window **32,000 tokens** hai.

Calculate karo (approx, 1 token ≈ 0.75 words English me):
1. Agar system prompt 500 tokens ka hai, aur tumhe ek document summarize karna hai jo 20,000 words ka hai — kya wo poora document ek hi request me fit ho jayega? Show karo calculation ke saath.
2. Agar nahi fit hota, toh kya solution hoga? (Hint: apne QR System project ke Redis session memory approach se connect karo — chunking bhi ek valid answer hai, agle Phase 3 me isko deeply padhoge)

---

## Task 4: Reflection — Apne Project Se Connect Karo (10 min)

Apne QR Food Ordering System ke agentic chatbot ke code me jao (jahan LLM API call ho raha hai):

1. Kya tumhare code me koi `temperature` parameter set kiya gaya hai? Value kya hai? Kyu wo value choose ki gayi thi (guess karo agar pata nahi, phir verify karo)?
2. Conversation history/Redis memory kaise LLM ko context window ke andar fit ki ja rahi hai — poora history bheja jata hai ya trimmed/summarized version?

**Deliverable:** Apna answer ek chhota paragraph me likho — ye interview me directly kaam aayega jab poochenge "apne project me LLM integration explain karo."