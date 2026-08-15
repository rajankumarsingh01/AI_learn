# Phase 1 → Chapter 4: Prompt Engineering Patterns — Practice Tasks

---

## Task 1: Zero-shot vs Few-shot Comparison (25 min)

Ek sentiment classification task lo — food delivery reviews.

1. **Zero-shot prompt** likho (bina examples ke), 5 different reviews test karo:
   - "Bahut tasty tha, jaldi aa gaya"
   - "Ekdum theek tha, kuch special nahi"
   - "Order galat aaya aur cold tha"
   - "Delivery time pe aaya but packaging kharab thi"
   - "Best food ever!"

2. **Few-shot prompt** banao (3 examples ke saath), same 5 reviews test karo.

3. Compare karo — consistency me kya farak dikha? (Especially mixed/ambiguous reviews jaise 4th wala)

**Deliverable:** Dono outputs ki table banao (Review | Zero-shot Output | Few-shot Output), observe karo kaha difference aaya.

---

## Task 2: Chain-of-Thought Test Karo (25 min)

Ye math/logic problem lo:

```
"Kaksha platform me 150 students enrolled hain. 
30% ne annual plan liya, 45% ne quarterly plan liya, 
baaki ne monthly plan liya. 
Monthly plan walo ki exact count kya hai?"
```

1. Without CoT — direct prompt do, output note karo.
2. With CoT — "step by step calculate karo" add karo, output note karo.
3. Ek aur complex problem khud banao (2-3 steps ka calculation) aur dono versions test karo.

**Deliverable:** Dono approaches ke outputs compare karo — kya CoT wale me accuracy/clarity better thi? Apna observation likho.

---

## Task 3: Role-Based Prompting Experiment (20 min)

Same question do, but do alag system prompts (roles) ke saath:

**Role A:** `"You are a strict technical interviewer. Be critical and point out flaws directly."`
**Role B:** `"You are an encouraging mentor helping a beginner. Be supportive and patient."`

Question dono ko: `"Review this code approach: using a for-loop to fetch data from database inside another for-loop (nested database calls)."`

**Deliverable:** Dono responses paste karo, observe karo tone/content me kitna farak aaya same underlying question ke liye.

---

## Task 4: Production-Ready Structured Prompt Banao (30 min)

Apne QR Food Ordering System ke order-extraction use-case ke liye, Structured Instruction Pattern (Role → Task → Constraints → Format → Examples) follow karke ek complete system prompt likho.

Requirements:
- Hinglish aur English dono input handle kare
- Quantity na bole toh default 1 maane
- Non-food conversation ignore kare
- JSON format me output de (Chapter 3 wala schema use karo)
- Kam se kam 2 few-shot examples include karo

Test karo is prompt ko in inputs pe:
1. `"Ek pizza chahiye"`
2. `"Aaj mausam kaisa hai?"` (non-food — check karo model ignore karta hai ya nahi)
3. `"Do samosa aur teen chai, jaldi bhejna"`

**Deliverable:** Poora system prompt + teeno test cases ke outputs. Ye directly tumhare actual project me use ho sakta hai agar improvement chahiye ho.