# Phase 2 → Chapter 1: Function Calling Concept

## Kya/Kyu/Kaise

**Kya hai ye?**
Function Calling (Tool Calling bhi kehte hain) wo mechanism hai jisse LLM sirf text generate karne tak limited nahi rehta — wo **decide** kar sakta hai ki kisi specific task ke liye ek "tool" (function) call karna zaroori hai, aur us function ko kaunse parameters ke saath call karna hai. Ye woh cheez hai jo ek "chatbot" ko "agent" banati hai.

**Kyu zaroori hai?**
LLM khud database query nahi kar sakta, khud API call nahi kar sakta, khud calculation reliably nahi kar sakta (large numbers me galti karta hai). Function calling isi gap ko bridge karta hai — LLM "reasoning/decision-making" karta hai, aur actual kaam (DB query, API call, calculation) tumhara normal code karta hai.

**Kaise kaam karta hai?**
Tum LLM ko available "tools" ka list dete ho (naam, description, expected parameters). User query aane par, LLM decide karta hai — "is task ke liye mujhe koi tool chahiye ya nahi, aur agar chahiye toh kaunsa, kaunse arguments ke saath." LLM khud function execute nahi karta — wo sirf bolta hai "ye function isse call karo," tumhara backend code actually usse execute karta hai, result wapas LLM ko deta hai, aur LLM us result ko use karke final response banata hai.

---

## 1. Poora Flow — Step by Step

Ye 5-step cycle samajhna sabse zaroori hai:

```
1. User query aati hai: "Mera order #4521 ka status kya hai?"

2. LLM ko available tools ka pata hota hai (tum ne define kiye the), 
   jaise: get_order_status(order_id)

3. LLM decide karta hai: "Is query ka jawab dene ke liye 
   get_order_status tool chahiye, order_id = 4521"
   → LLM ye "tool call request" return karta hai (khud call nahi karta)

4. Tumhara backend code ye request receive karta hai, 
   ACTUALLY get_order_status(4521) function ko call karta hai,
   database se result nikalta hai: { status: "Out for delivery", eta: "15 min" }

5. Ye result LLM ko wapas bheja jata hai, 
   LLM isse use karke natural language response banata hai:
   "Aapka order #4521 abhi delivery ke liye nikal chuka hai, 15 minute me pahunch jayega!"
```

**Critical point (interview me galat samjha jata hai):** LLM **kabhi khud function execute nahi karta**. Wo sirf ye decide karta hai "kaunsa function, kaunse arguments ke saath." Actual execution hamesha tumhare backend code ki responsibility hai. Ye security ke liye bhi zaroori hai — agar LLM khud arbitrary code execute kar sake, wo bahut bada security risk hoga.

---

## 2. Tool Definition — Kaisi Dikhti Hai

Tool ek JSON schema ke through define hoti hai — teen cheezein zaroori hain: naam, description, parameters.

```javascript
const tools = [
  {
    name: "get_order_status",
    description: "Get the current status and estimated delivery time of a food order by order ID",
    parameters: {
      type: "object",
      properties: {
        order_id: {
          type: "integer",
          description: "The unique order ID number"
        }
      },
      required: ["order_id"]
    }
  }
];
```

**`description` field ka mahatva:** Ye sabse important part hai jo log underestimate karte hain. LLM sirf naam se nahi, **description padh kar** decide karta hai kab ye tool use karna hai. Agar description vague hai, LLM galat situations me tool call kar sakta hai ya zaroori situations me miss kar sakta hai.

---

## 3. Model Kaise Decide Karta Hai Tool Use Karna Hai Ya Nahi

LLM ko diya jata hai:
1. User ka message
2. Available tools ki list (naam + description + parameters)

LLM internally (apne training se) samajhta hai — "kya is query ka answer sirf mere knowledge se de sakta hoon, ya mujhe real-time/specific data chahiye jo sirf ek tool provide kar sakta hai?"

### Example — Tool Zaroori Hai:
```
User: "Mera order #4521 kaha hai?"
→ LLM ko real-time order status chahiye, ye uske training data me nahi ho sakta
→ Tool call: get_order_status(order_id=4521)
```

### Example — Tool Zaroori Nahi Hai:
```
User: "Paneer tikka me kya ingredients hote hain?"
→ Ye general knowledge hai, LLM apne training data se hi answer de sakta hai
→ No tool call, direct text response
```

Ye distinguish karna hi LLM ka "reasoning" part hai function calling me.

---

## 4. QR System Connection — Apna Existing Code Samajhna

Tumhare QR Food Ordering System me function calling already implement hai. Typical tools jo wahan honge:

```javascript
const tools = [
  {
    name: "add_item_to_cart",
    description: "Add a food item with specified quantity to the user's cart",
    parameters: { /* item_name, quantity */ }
  },
  {
    name: "get_menu_items",
    description: "Fetch available menu items, optionally filtered by category",
    parameters: { /* category (optional) */ }
  },
  {
    name: "place_order",
    description: "Finalize and place the order with items currently in cart",
    parameters: { /* cart_id or user_id */ }
  }
];
```

Jab user bolta hai "menu dikhao" → LLM `get_menu_items` call karne ka decide karta hai. Jab user bolta hai "do samosa add karo" → LLM `add_item_to_cart` call karta hai with `item_name: "samosa", quantity: 2`.

**Redis session memory ka role yaha:** Cart state (kya-kya add hua hai abhi tak) Redis me maintain hoti hai, kyunki LLM khud koi state store nahi karta — har request stateless hai (Phase 1 Chapter 1 yaad karo).

---

## 5. Single Tool Call vs Multiple Tool Calls

Modern LLMs (jaise GPT-4o, Claude) ek hi response me **multiple tools parallel call** kar sakte hain agar zaroorat ho:

```
User: "Menu dikhao aur mera current cart bhi batao"

LLM decide karta hai: Do independent tools chahiye
→ Tool call 1: get_menu_items()
→ Tool call 2: get_cart_status(user_id=123)

Dono parallel execute hote hain, dono results LLM ko wapas jate hain,
LLM combined response banata hai.
```

Ye efficiency ke liye important hai — sequential tool calls (ek ke baad ek) se zyada latency lagti, parallel calls fast hote hain.

---

## Common Mistakes (Interview me pooche jate hain)

1. **"LLM khud function execute karta hai" — galat samajhna.** LLM sirf decide karta hai, execution hamesha backend code ki responsibility hai.
2. **Tool description ko vague/short likhna.** "Get order info" jaisa vague description LLM ko confuse karta hai kab use karna hai. Specific hona chahiye: "Get the current delivery status and ETA for a specific order using its order ID."
3. **Har chhoti cheez ke liye tool define karna.** Agar LLM apne knowledge se hi answer de sakta hai (jaise "recipe kaise banate hain"), tool ki zarurat nahi — unnecessary tools latency aur confusion badhate hain.
4. **Tool ke parameters ka type sahi define na karna.** Agar `order_id` ko `string` bataya but actually `integer` chahiye, downstream code me type mismatch errors aa sakte hain.