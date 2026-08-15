# Phase 2 → Chapter 3: Multi-Tool Selection & Chaining — Interview Q&A

---

**Q1. Parallel tool calls aur sequential tool calls (chaining) me kya difference hai?**

Parallel tool calls tab hote hain jab tools independent hon — dono ek hi response me request ho sakte hain, ek round-trip me execute hote hain, latency kam hoti hai. Sequential chaining tab zaroori hoti hai jab ek tool ka output dusre tool ka input ho — isme multiple round-trips lagti hain kyunki LLM ko pehle result dekhna padta hai next decision lene ke liye.

---

**Q2. Ek real example do jaha sequential chaining zaroori ho, single call se kaam na chale.**

"Mera sabse recent order cancel kar do" — pehle `get_recent_order` call karke order_id pata karna padega, phir usi order_id ko use karke `cancel_order` call karna padega. Single call se ye possible nahi kyunki order_id pehle se pata nahi hai, wo pehle tool ke result se hi milega.

---

**Q3. Tool chaining me error handling kyu extra critical hai?**

Kyunki agar chain ke beech ka step fail ho jaye (jaise order cancel fail hua), aur agla step (refund) us failure ko ignore karke aage badh jaye, toh inconsistent/wrong state ban sakti hai — jaise refund process ho jaye bina actual cancellation confirm kiye. Tool results me clear success/failure indication (`success: boolean`) hona chahiye taaki LLM chain rokne ya continue karne ka sahi decision le sake.

---

**Q4. Kab pata chalta hai ki tools bahut zyada fine-grained/granular ban gaye hain?**

Jab ek simple query ke liye 6-7+ sequential tool calls chahiye ho rahe hon. Agar kuch tools hamesha saath use hote hain (jaise get_order, get_payment_method, get_delivery_address), unhe ek combined tool (get_full_order_details) me merge karna better hota hai — latency aur complexity dono kam hoti hai.

---

**Q5. Conditional tool selection kya hoti hai, example do.**

Jab agle tool ka selection pichle result pe depend karta hai based on business logic — jaise order cancel karne ke baad, agar order abhi "preparing" status me tha (cancel-eligible), tabhi refund-eligibility check karo, warna nahi. LLM ye decisions system prompt me di gayi business rules ke basis pe leta hai.

---

**Q6. Multi-round tool calling me LLM poora plan pehle hi bana leta hai ya step-by-step decide karta hai?**

Default function-calling pattern me LLM step-by-step decide karta hai — har round me sirf agla immediate step, based on jo result abhi mila. Poora upfront planning (multi-step plan pehle hi bana lena) ek advanced pattern hai jo explicit planning frameworks (jaise LangGraph) me hota hai.

---

**Q7. Apne QR System me ek multi-step chain ka example do.**

"Menu dikhao, do samosa add karo, phir order place karo" — teen sequential tool calls: `get_menu_items()`, phir `add_item_to_cart(item_name, quantity)`, phir `place_order(cart_id)`. Redis session memory har round ke beech cart state maintain karti hai taaki place_order ko pata ho cart me kya add hua tha.

---

**Q8. Parallel-eligible tools ko galti se sequential design karna kyu problem hai?**

Kyunki isse unnecessary latency add hoti hai — agar do tools genuinely independent hain (ek dusre pe depend nahi karte), unhe parallel call allow karna chahiye. Artificially sequential banane se do round-trips lagti hain jaha ek hi kaafi thi.

---

**Q9. Chain me maximum length/timeout limit rakhna kyu zaroori hai production me?**

Kyunki agar LLM kisi rare edge case me repetitive/looping decisions le (jaise ek hi tool baar-baar call kare bina progress ke), bina limit ke ye resources waste karega, latency badhayega, aur cost bhi badhayega. Production safety ke liye max-rounds ya timeout enforce karna best practice hai.

---

**Q10. Tool result me `success: boolean` field jaisi cheez kyu design pattern hai, sirf data return karna kaafi kyu nahi?**

Kyunki LLM ko explicit signal chahiye hota hai chain continue karni hai ya rokni hai. Agar sirf data return ho aur success/failure ambiguous ho, LLM galat interpret kar sakta hai ki operation successful hua jabki actually fail hua tha, jisse chain me galat aage ke steps execute ho sakte hain.