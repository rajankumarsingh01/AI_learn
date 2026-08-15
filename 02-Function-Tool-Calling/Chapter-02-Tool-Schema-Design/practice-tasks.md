# Phase 2 → Chapter 2: Tool Schema Design — Practice Tasks

---

## Task 1: Weak Schema Ko Improve Karo (20 min)

Neeche di gayi weak schema ko improve karo — naming, description, enum, aur parameter descriptions sab better banao:

```json
{
  "name": "update",
  "description": "Updates something",
  "parameters": {
    "type": "object",
    "properties": {
      "id": { "type": "string" },
      "status": { "type": "string" }
    },
    "required": ["id"]
  }
}
```

Context: Ye tool order ka status update karne ke liye hai (pending → preparing → delivered → cancelled).

**Deliverable:** Improved schema likho, aur likho kya-kya changes kiye aur kyu (naming, description, enum, parameter names).

---

## Task 2: Disambiguation Test Karo — Real API Se (30 min)

In do tools ko ek saath define karo (bina disambiguation ke pehle):

```javascript
const tools = [
  {
    name: "get_status",
    description: "Get status",
    input_schema: { type: "object", properties: { id: { type: "string" } }, required: ["id"] }
  },
  {
    name: "get_info",
    description: "Get info",
    input_schema: { type: "object", properties: { id: { type: "string" } }, required: ["id"] }
  }
];
```

Query bhejo: `"Order #123 ka status kya hai?"` — dekho model kaunsa tool choose karta hai (agar dono vague hain, confusion ho sakta hai).

Ab dono tools ki descriptions improve karo (clear disambiguation ke saath), same query firse test karo.

**Deliverable:** Before/after comparison — kya vague descriptions se galat/inconsistent tool selection hua? Improved version me kya better hua?

---

## Task 3: Enum Validation Test Karo (20 min)

Ek tool banao jisme `order_status_filter` parameter enum ke saath ho (notes.md wala example use karo).

Test karo 3 different phrasings ke saath:
1. `"Mujhe pending orders dikhao"`
2. `"Jo orders abhi tak deliver nahi hue"`
3. `"Cancelled wale orders dikhao"`

Verify karo — kya model teeno cases me exact enum values (`"pending"`, `"delivered"` ka opposite logic, `"cancelled"`) hi use karta hai, ya kabhi variation deta hai?

**Deliverable:** Teeno test cases ke actual tool-call outputs paste karo, verify karo enum consistency maintain hui.

---

## Task 4: Apne QR System Ki Schemas Ko Redesign Karo (30 min)

Apne QR System ke actual tools (jo Task 4, Chapter 1 me audit kiye the) ko is chapter ke principles se dobara review karo:

1. Kya sab descriptions "formula" (kya karta hai + kab use karna hai + kya return karta hai) follow karti hain?
2. Kya kahi enum use hona chahiye tha but string free-text rakha gaya hai?
3. Kya koi parameter naam ambiguous hai (generic `id`, `value`, `data` jaisa)?
4. Kya similar-sounding tools ke beech disambiguation clear hai?

**Deliverable:** Ek-do tools ki improved/redesigned schema likho (agar improvement ki zarurat mili), ya agar already achhi hai toh likho "ye schema already achhi practice follow karti hai kyunki..." — dono cases interview me useful hain.