# Phase 2 → Chapter 1: Function Calling Concept — Interview Q&A

---

**Q1. Function calling kya hai, simple words me samjhao?**

Function calling wo mechanism hai jisse LLM decide kar sakta hai ki kisi user query ka jawab dene ke liye ek specific "tool" (function) call karna zaroori hai, aur kaunse parameters ke saath. Ye LLM ko sirf text-generator se "agent" banata hai jo real actions trigger kar sakta hai.

---

**Q2. Kya LLM khud function execute karta hai? Ye clarify karo.**

Nahi, ye sabse common misconception hai. LLM sirf decide karta hai "kaunsa function, kaunse arguments ke saath call karna hai" aur ye request return karta hai. Actual function execution hamesha backend code ki responsibility hoti hai — security aur control ke liye ye zaroori separation hai.

---

**Q3. Function calling ka poora flow 5 steps me samjhao.**

(1) User query aati hai. (2) LLM ko available tools pata hote hain. (3) LLM decide karta hai kaunsa tool chahiye aur kaunse arguments ke saath, ye request return karta hai. (4) Backend code actually us function ko execute karta hai (DB query, API call, etc.) aur result nikalta hai. (5) Result LLM ko wapas bheja jata hai, jisse LLM natural language me final response banata hai.

---

**Q4. Tool definition me `description` field itna important kyu hai?**

Kyunki LLM sirf function name se nahi, description padh kar decide karta hai kab wo tool use karna hai. Vague description ("get order info") se LLM confuse ho sakta hai — galat situations me call kar sakta hai ya zaroori situations me miss kar sakta hai. Specific, clear description accurate tool-selection ensure karta hai.

---

**Q5. LLM kaise decide karta hai ki tool call karna zaroori hai ya nahi?**

LLM internally judge karta hai ki query ka answer uske training data/general knowledge se diya ja sakta hai, ya real-time/specific/user-specific data chahiye jo sirf ek tool provide kar sakta hai. Jaise "order status kya hai" ke liye tool zaroori hai (real-time data), lekin "recipe kaise banate hain" ke liye tool zaroori nahi (general knowledge).

---

**Q6. Apne QR System project me tool calling kaise kaam karta hai, example do.**

Jab user "menu dikhao" bolta hai, LLM `get_menu_items` tool call karne ka decide karta hai. Jab user "do samosa add karo" bolta hai, LLM `add_item_to_cart` tool call karta hai with parameters `item_name: "samosa", quantity: 2`. Backend actual cart update karta hai, result LLM ko wapas jata hai jo confirmation message banata hai.

---

**Q7. Cart state (jo user ne add kiya hai) kaha maintain hoti hai, aur LLM khud kyu nahi rakhta?**

Redis session memory me maintain hoti hai. LLM khud koi persistent state store nahi karta kyunki har API request stateless hoti hai (Phase 1 se concept) — LLM ke paas koi memory nahi hoti requests ke beech. Isliye application-level state (cart, session data) explicitly database/Redis me store karna padta hai.

---

**Q8. Multiple tools ek hi response me parallel call ho sakte hain kya? Example do.**

Haan, modern LLMs (GPT-4o, Claude) parallel tool calls support karte hain jab independent tools chahiye ho ek saath. Jaise "menu dikhao aur mera cart bhi batao" — is query ke liye LLM `get_menu_items()` aur `get_cart_status()` dono ek saath call kar sakta hai, sequential ki bajaye — isse latency kam hoti hai.

---

**Q9. Har chhote task ke liye tool define karna kyu problematic hai?**

Kyunki agar LLM apne general knowledge se hi accurately answer de sakta hai (jaise "kya ye dish vegetarian hai" agar clear naam se pata chal jaye), toh unnecessary tool definition LLM ko confuse kar sakta hai kab use karna hai, aur extra latency/complexity add karta hai bina real benefit ke.

---

**Q10. Agar tool parameter ka type galat define kar diya (string ki jagah integer expect ho raha hai), kya problem ho sakti hai?**

Downstream backend code me type mismatch error aa sakta hai jab LLM ka diya hua argument directly function ko pass kiya jaye — jaise agar `order_id` string "4521" aaye lekin database query integer expect kare, toh query fail ho sakti hai ya unexpected behavior aa sakta hai. Isliye schema me correct types define karna zaroori hai.