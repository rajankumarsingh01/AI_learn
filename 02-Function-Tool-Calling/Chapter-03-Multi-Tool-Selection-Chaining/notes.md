# Phase 2 → Chapter 3: Multi-Tool Selection & Chaining

## Kya/Kyu/Kaise

**Kya hai ye?**
Ab tak humne single tool calls dekhe. Real production systems me LLM ko kabhi ek query solve karne ke liye **multiple tools** use karne padte hain — kabhi parallel (ek saath), kabhi sequential (ek tool ka output dusre tool ka input banta hai — isse "chaining" kehte hain).

**Kyu zaroori hai?**
Real user queries simple nahi hoti. "Mera order cancel karke refund process karo" — isme do actions chahiye: (1) order cancel karna, (2) refund initiate karna, aur dusra pehle wale ke result pe depend karta hai. Agar tumhe pata nahi chaining kaise design karte hain, tumhare agent complex multi-step tasks handle nahi kar payenge — jo real agentic systems (jaise tumhara SentinelAI) ka core requirement hai.

**Kaise kaam karta hai?**
LLM ek response me multiple tool calls request kar sakta hai (agar independent ho — parallel), ya ek tool ka result dekhne ke baad decide karta hai agla tool kaunsa call karna hai (sequential — isme LLM ko multiple "rounds" chahiye hote hain, har round me ek tool result milta hai).

---

## 1. Parallel Tool Calls — Jab Tools Independent Hon

Agar do tools ek dusre pe depend nahi karte, LLM unhe ek hi response me parallel request kar sakta hai.

```
User: "Mera cart dikhao aur ye bhi batao restaurant abhi open hai ya nahi"

LLM analysis: Do independent pieces of info chahiye
→ Tool call 1: get_cart_status(user_id=123)
→ Tool call 2: check_restaurant_open_status(restaurant_id=45)

Dono ek saath backend ko bheje jate hain, dono execute hote hain,
dono results LLM ko wapas jate hain ek hi follow-up request me.
```

**Performance benefit:** Agar sequential karte (pehle cart, phir status), do round-trips lagti. Parallel me ek hi round-trip me dono ho jate hain — latency kam.

---

## 2. Sequential Tool Calls (Chaining) — Jab Ek Result Dusre Ka Input Ho

Jab ek tool ka output next tool ke input ke liye zaroori ho, LLM ko sequential rounds chahiye:

```
User: "Mera sabse recent order cancel kar do"

Round 1:
LLM decide karta hai: pehle "sabse recent order" ID pata karni hogi
→ Tool call: get_recent_order(user_id=123)
→ Result: { order_id: 4521, status: "preparing" }

Round 2:
LLM ab is order_id ko use karke agla decision leta hai:
→ Tool call: cancel_order(order_id=4521)
→ Result: { success: true, message: "Order cancelled" }

Round 3:
LLM final natural language response banata hai:
"Aapka order #4521 cancel ho gaya hai."
```

**Important:** Ye 3 separate API round-trips hain (LLM ↔ backend). Har round me LLM sirf agla step decide karta hai based on jo result abhi mila — wo pehle se poora plan nahi banata (jab tak explicit planning pattern na ho, jo LangGraph me Phase 6 me padhoge).

---

## 3. Conditional Tool Selection — Result Ke Basis Pe Decision

Kabhi agle tool ka selection pichle result pe depend karta hai:

```
User: "Order cancel karke agar eligible ho toh refund bhi process karo"

Round 1: get_order_details(order_id=4521)
→ Result: { status: "preparing", payment_method: "UPI", amount: 450 }

Round 2: LLM check karta hai — "preparing" status me cancel allowed hai (business logic 
LLM ko system prompt me batayi gayi hogi)
→ Tool call: cancel_order(order_id=4521)
→ Result: { success: true }

Round 3: LLM decide karta hai — cancel successful hua, ab refund eligibility check karo
→ Tool call: check_refund_eligibility(order_id=4521)
→ Result: { eligible: true, refund_amount: 450 }

Round 4: Eligible hai, toh refund process karo
→ Tool call: process_refund(order_id=4521, amount=450)
→ Result: { success: true, refund_id: "RF789" }

Round 5: Final response:
"Order cancel ho gaya aur ₹450 ka refund process ho gaya hai (Refund ID: RF789)."
```

Ye ek **4-tool chain** hai jisme har step pichle step ke result pe depend karta hai. Ye pattern hi agentic behavior ka core hai — LLM dynamically decide kar raha hai next step kya hoga, based on real data.

---

## 4. Error Handling In Chains — Kya Ho Agar Beech Me Kuch Fail Ho

Chaining me error handling extra critical hai — agar step 2 fail ho jaye, step 3/4 ka kya hoga?

```
Round 2: cancel_order(order_id=4521)
→ Result: { success: false, error: "Order already out for delivery, cannot cancel" }

Round 3: LLM ko ye failure result milta hai, wo:
- Refund process NAHI karega (chain ruk jani chahiye)
- User ko explain karega kya hua:
"Maaf kijiye, ye order already delivery ke liye nikal chuka hai, isliye cancel nahi ho sakta."
```

**Design principle:** Tool results me hamesha clear success/failure indication honi chahiye (jaise `success: boolean` field), taaki LLM ko pata chale chain continue karni hai ya rokni hai.

---

## 5. QR System Connection — Real Chaining Example

Tumhare QR System me ye chain ho sakti hai:

```
User: "Menu dikhao, do samosa add karo, phir order place kar do"

Round 1: get_menu_items() → menu data aata hai, LLM confirms samosa available hai
Round 2: add_item_to_cart(item_name="samosa", quantity=2) → cart update hota hai
Round 3: place_order(cart_id=...) → order create hota hai
Round 4: Final response: "Aapka order place ho gaya hai — 2 samosa!"
```

Redis session memory yaha crucial role play karti hai — har round ke beech cart state maintain honi chahiye, taaki round 3 ko pata ho round 2 me kya add hua tha.

---

## 6. Kab Chaining "Too Much" Ho Jati Hai — Design Caution

Agar ek query ke liye 6-7+ sequential tool calls chahiye ho rahe hain, ye sign hai ki:
- Ya toh task genuinely bahut complex hai (fine, but latency ka dhyan rakho)
- Ya tumhare tools bahut granular/fine-grained hain — unhe combine karna chahiye tha ek bade tool me

**Example:** Agar `get_order`, `get_payment_method`, `get_delivery_address` teen separate tools hain jo hamesha saath use hote hain, unhe combine karke ek `get_full_order_details` tool banana better ho sakta hai — latency kam hogi.

---

## Common Mistakes (Interview me pooche jate hain)

1. **Parallel-eligible tools ko sequentially design karna** — agar do tools independent hain, unhe parallel-callable rehne dena chahiye, artificially dependency create mat karo.
2. **Chain me error propagation na sochna** — agar beech ka step fail ho, aur agla step us failure ko ignore karke chal jaye, wrong/inconsistent state ban sakti hai (jaise refund process ho jaye bina actual cancellation confirm kiye).
3. **Bahut fine-grained tools banana** — isse har simple task ke liye lambi chain chahiye hoti hai, latency aur complexity dono badhti hai.
4. **Chain ki koi max-length/timeout limit na rakhna** — agar LLM kisi loop me phas jaye (rare but possible), production me infinite/bahut lambi chain se resources waste ho sakte hain.