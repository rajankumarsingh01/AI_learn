# Phase 1 → Chapter 4: Prompt Engineering Patterns

## Kya/Kyu/Kaise

**Kya hai ye?**
Prompt engineering ka matlab hai — LLM ko input dene ka tareeka is tarah design karna ki tumhe consistently better, accurate, aur predictable output mile. Ye "sirf sawal poochna" nahi hai — ye ek engineering skill hai, kyunki same task ke liye alag-alag prompt structure se result dramatically different aa sakta hai.

**Kyu zaroori hai?**
Agar tum LangChain/LangGraph seedha seekh lo bina prompt engineering samjhe, toh tumhare AI agents unreliable honge — kabhi sahi jawab denge, kabhi galat format me, kabhi hallucinate karenge. Prompt engineering hi wo foundation hai jispe RAG, agents, sab kuch tika hota hai. Interview me bhi directly poocha jata hai: "apne project me prompt kaise design kiya?"

**Kaise kaam karta hai?**
LLM next-token prediction karta hai (Chapter 1 yaad karo) — iska matlab hai jo pattern tum prompt me dikhaoge, model wahi pattern continue karne ki koshish karega. Prompt engineering patterns is fact ka fayda uthate hain — tum LLM ko examples, structure, ya reasoning steps dikha ke uska behavior "guide" karte ho.

---

## 1. Zero-Shot Prompting

Sabse basic pattern — koi example diye bina seedha task bata do.

```
Prompt: "Classify this review as positive or negative: 'Food was cold and delivery was late.'"

Output: "Negative"
```

**Kab use karo:** Simple, common tasks jo model already achhe se samajhta hai (basic classification, translation, summarization).

**Limitation:** Complex ya domain-specific tasks me consistency kam hoti hai kyunki model ko pata nahi tumhe exact kaunsa format/style chahiye.

---

## 2. Few-Shot Prompting

Model ko 2-3 examples dikhao taaki wo pattern samajh jaye — format, tone, style sab.

```
Prompt:
"Classify the sentiment of these food delivery reviews:

Review: 'Amazing taste, fast delivery!'
Sentiment: Positive

Review: 'Order was wrong and cold.'
Sentiment: Negative

Review: 'Okay food, nothing special.'
Sentiment: Neutral

Review: 'Delivery boy was rude and food spilled.'
Sentiment:"

Output: "Negative"
```

**Kyu better hai zero-shot se:** Examples dikhane se model ko exact output format (Positive/Negative/Neutral — capital P, single word) samajh aata hai, consistency badhti hai.

**QR System connection:** Agar tumhare chatbot ko specific tone/style me reply karna hai (jaise Hinglish, friendly), few-shot examples system prompt me dena best practice hai.

---

## 3. Chain-of-Thought (CoT) Prompting

Model ko explicitly bolo "step-by-step socho" — complex reasoning tasks me accuracy dramatically improve hoti hai.

### Without CoT:
```
Prompt: "A restaurant has 45 orders. 12 are cancelled, 8 are refunded (different from cancelled). 
How many orders are actually being prepared?"

Output: "25" (galat ho sakta hai — model directly jump kar sakta hai answer pe)
```

### With CoT:
```
Prompt: "A restaurant has 45 orders. 12 are cancelled, 8 are refunded (different from cancelled). 
How many orders are actually being prepared? Think step by step."

Output: 
"Step 1: Total orders = 45
Step 2: Cancelled orders = 12, so remaining = 45 - 12 = 33
Step 3: Refunded orders = 8 (separate from cancelled), so remaining = 33 - 8 = 25
Answer: 25 orders are being prepared."
```

Dono ka answer same hai is example me, lekin **complex multi-step problems me CoT accuracy ko significantly improve karta hai** kyunki model ko "jaldi jawab dene" ki bajaye reasoning process follow karna padta hai.

**Interview-relevant fact:** Modern reasoning models (jaise o1, Claude ke extended thinking mode) internally hamesha CoT-jaisa process follow karte hain, lekin normal chat models me explicit "think step by step" instruction dena still helpful hai.

---

## 4. Role-Based Prompting

Model ko ek specific "persona" ya role assign karna — response ka tone/expertise level change karta hai.

```
System prompt: "You are a senior backend engineer reviewing code for security vulnerabilities. 
Be direct, technical, and point out issues without sugarcoating."
```

vs

```
System prompt: "You are a friendly customer support assistant for a food delivery app. 
Be warm, empathetic, and use simple language."
```

**Kyu matter karta hai:** Same underlying model, but role define karne se output style, vocabulary, aur focus dramatically change hote hain. Ye tumhare SentinelAI project ke liye bhi relevant hai — "security auditor agent" ka role clearly define karna hoga.

---

## 5. Structured Instruction Pattern (Best Practice for Production)

Production prompts me clear structure follow karna chahiye — random paragraph nahi likhna:

```
System prompt:

## Role
You are an order-extraction assistant for a food delivery app.

## Task
Extract food items and quantities from user messages in Hinglish or English.

## Constraints
- Only extract items explicitly mentioned
- If quantity not mentioned, default to 1
- Ignore any non-food related conversation

## Output Format
Return JSON: { "items": [{"name": string, "quantity": number}] }

## Examples
Input: "Do samosa chahiye"
Output: {"items": [{"name": "samosa", "quantity": 2}]}
```

Ye structure (Role → Task → Constraints → Format → Examples) production prompts me widely use hota hai kyunki ye readable bhi hai aur model ke liye follow karna bhi easy hai.

---

## 6. Negative Prompting (Kya NAHI karna hai, batana)

Sirf positive instructions kaafi nahi hote — kabhi explicit "mat karo" bolna zaroori hai.

```
"Answer the user's question about their order.
Do NOT make up information about delivery times if you don't have real data.
Do NOT discuss topics unrelated to food ordering."
```

**Kyu important hai:** Bina negative constraints ke, model kabhi kabhi "helpful lagne ki koshish" me galat/fabricated info de deta hai (hallucination) ya off-topic ho jata hai.

---

## Common Mistakes (Interview me pooche jate hain)

1. **Vague instructions dena** — "Achha jawab do" jaisa prompt useless hai. Specific, measurable instructions do ("2-3 sentences me answer do, formal tone use karo").
2. **Examples ka format inconsistent rakhna** — Few-shot me agar examples ka format khud hi consistent nahi hai, model confuse ho jayega.
3. **CoT ko simple tasks pe bhi force karna** — Chhote, straightforward tasks pe "think step by step" bolna sirf extra tokens/latency add karta hai, benefit nahi deta.
4. **System prompt bahut lamba aur unorganized rakhna** — Structure follow na karne se model important constraints miss kar sakta hai. Structured format (Role/Task/Constraints/Format) use karo.