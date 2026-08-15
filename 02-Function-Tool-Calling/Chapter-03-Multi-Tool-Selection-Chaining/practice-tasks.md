# Phase 2 → Chapter 3: Multi-Tool Selection & Chaining — Practice Tasks

---

## Task 1: Parallel Tool Calls Test Karo (25 min)

Do independent tools define karo aur real API call se test karo ki model parallel call karta hai:

```javascript
const tools = [
  {
    name: "get_cart_status",
    description: "Get the current items in the user's cart",
    input_schema: {
      type: "object",
      properties: { user_id: { type: "integer" } },
      required: ["user_id"]
    }
  },
  {
    name: "check_restaurant_open_status",
    description: "Check if the restaurant is currently open for orders",
    input_schema: {
      type: "object",
      properties: { restaurant_id: { type: "integer" } },
      required: ["restaurant_id"]
    }
  }
];

// Query: "Mera cart dikhao aur ye bhi batao restaurant open hai ya nahi"
```

Response me check karo — `content` array me kitne `tool_use` blocks hain? Agar 2 hain ek hi response me, parallel calling confirm ho gayi.

**Deliverable:** Response ka `content` array paste karo, dikhao dono tool_use blocks present hain.

---

## Task 2: Full Sequential Chain Implement Karo (40 min)

Notes.md wala "recent order cancel karo" example ko poora implement karo — fake functions ke saath:

```javascript
function getRecentOrder(userId) {
  return { order_id: 4521, status: "preparing" };
}

function cancelOrder(orderId) {
  return { success: true, message: "Order cancelled" };
}

const tools = [
  {
    name: "get_recent_order",
    description: "Get the user's most recent order details",
    input_schema: {
      type: "object",
      properties: { user_id: { type: "integer" } },
      required: ["user_id"]
    }
  },
  {
    name: "cancel_order",
    description: "Cancel an order using its order ID",
    input_schema: {
      type: "object",
      properties: { order_id: { type: "integer" } },
      required: ["order_id"]
    }
  }
];

// Multi-round loop banao:
// Round 1: query bhejo "Mera sabse recent order cancel kar do"
// Model get_recent_order call karega
// Fake function se result lo, wapas bhejo
// Model ab cancel_order call karega (order_id use karke jo pehle mila)
// Fake function se result lo, wapas bhejo
// Model final natural language response dega
```

Poora conversation flow implement karo (jaisa Phase 2 Chapter 1 Task 3 me kiya tha, lekin do rounds ke saath).

**Deliverable:** Poora console output — dikhao model ne sahi order_id use kiya round 1 se round 2 me, aur final response sahi bana.

---

## Task 3: Error Propagation Test Karo (25 min)

Task 2 wale `cancelOrder` function ko modify karo taaki wo failure simulate kare:

```javascript
function cancelOrder(orderId) {
  return { success: false, error: "Order already out for delivery, cannot cancel" };
}
```

Poora chain firse run karo — verify karo:
1. Model ye failure result ko sahi se samajhta hai
2. Model koi next step (jaise refund) attempt NAHI karta agar wo chain me hota
3. Final response me user ko clearly bataya jata hai ki cancel kyu fail hua

**Deliverable:** Console output jisme dikhe failure gracefully handled hua, chain aage nahi badhi.

---

## Task 4: Apne QR System Ki Ek Real Chain Map Karo (25 min)

Apne QR System me dhundo ek real multi-step user flow (jaise "menu dikhao → item add karo → order place karo"):

1. Ye kaunse tools involve karti hai, sequence me likho.
2. Kya ye currently parallel-eligible steps ko sequential treat kar rahi hai (agar koi optimization possible hai)?
3. Kya error handling hai agar beech ka koi step fail ho (jaise item out-of-stock nikle add karte waqt)?
4. Redis session memory kaise ensure karti hai ki chain ke beech state maintain rahe?

**Deliverable:** Apni actual chain ka step-by-step breakdown likho — ye interview me directly bol sakte ho "mere agent ka multi-step flow is tarah kaam karta hai."