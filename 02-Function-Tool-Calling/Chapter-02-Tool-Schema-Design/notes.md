# Phase 2 → Chapter 2: Tool Schema Design

## Kya/Kyu/Kaise

**Kya hai ye?**
Chapter 1 me humne dekha tool kya hoti hai. Ab is chapter me dekhenge ki **achha tool schema kaise design karte hain** — taaki LLM consistently sahi tool chune, sahi arguments extract kare, aur galtiyan kam ho. Ye ek "engineering skill" hai — same functionality, lekin ek achhi schema vs ek buri schema ka real-world accuracy me bahut farak padta hai.

**Kyu zaroori hai?**
Chapter 1 me sirf "tool kaisi dikhti hai" dekha. Lekin production me agar schema design weak hai — jaise ambiguous parameter names, missing descriptions, wrong types — toh LLM galat tool choose karega, ya sahi tool but galat arguments ke saath call karega. Ye woh gap hai jo "tutorial-level function calling" aur "production-grade function calling" ko differentiate karta hai.

**Kaise design karte hain?**
Achhi schema design ke kuch concrete principles hote hain — naming conventions, description-writing patterns, parameter constraints (enums, required vs optional), aur multiple-tools-ke-beech disambiguation. Ye chapter in sab patterns ko cover karta hai with examples.

---

## 1. Naming Conventions — Clear Aur Consistent Rakho

Tool names verb-noun pattern follow karne chahiye, aur poore system me consistent naming style honi chahiye.

### Achha:
```
get_order_status
add_item_to_cart
update_delivery_address
cancel_order
```

### Bura (avoid karo):
```
orderStatus       // inconsistent casing style
CartAdd           // unclear verb-noun order
delivery_update   // ambiguous kya update ho raha hai
process           // bahut generic, kuch bhi ho sakta hai
```

**Kyu matter karta hai:** LLM naam se bhi context leta hai (description ke alawa). Consistent, descriptive naming se model ko pattern samajhna easy hota hai, especially jab tumhare paas 10-15+ tools ho.

---

## 2. Description Writing — Ek Formula

Har tool description me ye 3 cheezein honi chahiye:
1. **Kya karta hai** (core action)
2. **Kab use karna hai** (trigger condition, agar non-obvious ho)
3. **Kya return karta hai** (agar helpful ho disambiguation ke liye)

### Weak description:
```json
{
  "name": "get_order_status",
  "description": "Gets order status"
}
```

### Strong description:
```json
{
  "name": "get_order_status",
  "description": "Retrieves the current status (preparing, out for delivery, delivered, cancelled) and estimated delivery time for a specific order. Use this when the user asks about the progress or location of an existing order they've placed."
}
```

**Farak dikha?** Strong version LLM ko batata hai exact kab use karna hai ("existing order" — naya order place karne ke liye nahi), aur kya milega (status categories bhi hint kiye).

---

## 3. Parameter Design — Types, Enums, Required/Optional

### Enums use karo jaha limited valid values hon:
```json
{
  "properties": {
    "order_status_filter": {
      "type": "string",
      "enum": ["pending", "preparing", "delivered", "cancelled"],
      "description": "Filter orders by their current status"
    }
  }
}
```

**Kyu:** Bina enum ke, LLM kabhi "Pending" (capital P), kabhi "pending" bhej sakta hai — inconsistency. Enum se model ko **exact valid values** pata hote hain, wo unhi me se choose karega.

### Required vs Optional clearly define karo:
```json
{
  "properties": {
    "item_name": { "type": "string", "description": "Name of the food item" },
    "quantity": { "type": "integer", "description": "Number of units, defaults to 1 if not specified" }
  },
  "required": ["item_name"]
}
```

Yaha `quantity` optional hai (required array me nahi hai) — description me default value bhi mention ki gayi hai, taaki LLM samjhe agar user quantity na bole toh kya assume karna hai.

### Descriptions individual parameters ke liye bhi likho:
Sirf tool-level description kaafi nahi — har parameter ka bhi apna description hona chahiye, especially agar naam se clear na ho.

```json
{
  "properties": {
    "id": {
      "type": "integer"
    }
  }
}
```

Upar wala BAD hai — naam se pata nahi kaunsa ID (order? user? item?). Neeche wala GOOD hai:

```json
{
  "properties": {
    "order_id": {
      "type": "integer",
      "description": "The unique numeric identifier of the order (not the user ID or item ID)"
    }
  }
}
```

---

## 4. Multiple Tools Ke Beech Disambiguation

Jab tumhare paas similar-sounding tools hon, explicit disambiguation likhna zaroori hai description me:

```json
[
  {
    "name": "get_order_status",
    "description": "Get status of an EXISTING order that has already been placed. Use for questions like 'where is my order' or 'has my order shipped'."
  },
  {
    "name": "get_menu_items",
    "description": "Fetch available food items from the menu that CAN BE ORDERED. Use for questions like 'what can I order' or 'show me the menu'. This is NOT for checking existing order status."
  }
]
```

Notice "This is NOT for..." wala pattern — explicitly bolna ki ye tool kis cheez ke liye NAHI hai, confusion kam karta hai jab tools thoda overlap-sounding hon.

---

## 5. Nested Objects Aur Arrays — Complex Schemas

Kabhi kabhi ek tool ko complex/nested data chahiye hoti hai:

```json
{
  "name": "place_order",
  "description": "Place a new order with the specified items and delivery details",
  "parameters": {
    "type": "object",
    "properties": {
      "items": {
        "type": "array",
        "description": "List of items to order",
        "items": {
          "type": "object",
          "properties": {
            "item_name": { "type": "string" },
            "quantity": { "type": "integer" }
          },
          "required": ["item_name", "quantity"]
        }
      },
      "delivery_address": {
        "type": "string",
        "description": "Full delivery address"
      }
    },
    "required": ["items", "delivery_address"]
  }
}
```

Ye QR System ke `place_order` jaise tool ka real structure ho sakta hai — array of objects, jisme har object ka apna schema hai.

---

## 6. Schema Testing — Edge Cases Check Karo

Production me deploy karne se pehle in cases ko test karna chahiye:
1. **Ambiguous input:** "Kuch order karna hai" (specific item nahi bola) — kya model sahi se clarification maangta hai ya galat guess karta hai?
2. **Multiple valid interpretations:** "Do samosa aur ek chai, aur wo bhi jaldi bhejna" — kya "jaldi" ko koi galat parameter me daal deta hai?
3. **Missing required info:** Agar `delivery_address` nahi diya user ne — kya tool call fail hota hai gracefully ya LLM khud assume kar leta hai (bura sign)?

---

## Common Mistakes (Interview me pooche jate hain)

1. **Description sirf ek line, generic likhna** — "Gets data" jaisa description kaafi vague hai, LLM ko decision-making ke liye kaafi context nahi milta.
2. **Enum use na karna jaha valid values limited hon** — isse LLM inconsistent string values generate karta hai jo backend validation fail kar sakti hain.
3. **Similar-naam wale tools ke beech disambiguation na likhna** — jab tools overlap-sounding hon, model confuse hoke galat tool choose kar sakta hai.
4. **Parameter names ambiguous rakhna** (id, value, data jaise generic naam) — specific naam use karo (order_id, item_quantity).