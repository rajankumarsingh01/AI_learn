# Phase 2 → Chapter 4: Error Handling — Bad Tool Calls — Practice Tasks

---

## Task 1: Missing Argument Scenario Simulate Karo (25 min)

Ek `cancel_order` tool banao aur is query ko test karo: `"Mera order cancel kar do"` (bina order ID diye).

```javascript
function handleToolCall(toolName, toolInput) {
  if (toolName === "cancel_order") {
    if (!toolInput.order_id) {
      return { success: false, error: "order_id is missing. Ask the user which order they want to cancel." };
    }
    return { success: true, message: "Order cancelled" };
  }
}
```

Poora flow implement karo — LLM ko tool call karne do, tumhara validation missing order_id catch kare, error result LLM ko wapas bhejo, aur dekho LLM kya follow-up response deta hai.

**Deliverable:** Console output dikhao — LLM ko missing argument ka error result mila, aur usne user se clarification maangi (ya nahi — agar nahi maangi, note karo ye ek issue hai).

---

## Task 2: External Failure Simulate Karo (25 min)

Ek fake database function banao jo random fail ho:

```javascript
async function fakeDatabaseCancelOrder(orderId) {
  const shouldFail = Math.random() < 0.5; // 50% chance of failure
  if (shouldFail) {
    throw new Error("Connection timeout to database");
  }
  return { cancelled: true, orderId };
}

async function handleToolCall(toolName, toolInput) {
  if (toolName === "cancel_order") {
    try {
      const result = await fakeDatabaseCancelOrder(toolInput.order_id);
      return { success: true, data: result };
    } catch (error) {
      return { success: false, error: "Unable to process cancellation right now. Please try again in a moment." };
    }
  }
}
```

Function ko 3-4 baar run karo (kabhi success, kabhi fail aayega random se). Verify karo:
1. Raw error message ("Connection timeout to database") LLM ko kabhi nahi jata
2. LLM ko sirf clean, user-friendly error milta hai
3. LLM appropriately deals karta hai dono cases (success aur failure) me

**Deliverable:** Dono scenarios (success aur failure) ke console outputs paste karo.

---

## Task 3: Validation Layer Banao Aur Test Karo (30 min)

Ek complete validation function banao `place_order` tool ke liye (jisme `items` array aur `delivery_address` chahiye):

```javascript
function validatePlaceOrderInput(input) {
  const errors = [];
  
  if (!input.items || !Array.isArray(input.items) || input.items.length === 0) {
    errors.push("At least one item is required");
  }
  
  if (input.items) {
    input.items.forEach((item, idx) => {
      if (!item.item_name) errors.push(`Item ${idx + 1} is missing item_name`);
      if (!item.quantity || item.quantity < 1) errors.push(`Item ${idx + 1} has invalid quantity`);
    });
  }
  
  if (!input.delivery_address || input.delivery_address.trim() === "") {
    errors.push("Delivery address is required");
  }
  
  return { valid: errors.length === 0, errors };
}
```

Test karo in cases ke saath:
1. Valid input — sab kuch sahi
2. Empty items array
3. Item ke saath quantity 0 ya negative
4. Missing delivery_address

**Deliverable:** Har case ka validation output paste karo, verify karo errors sahi catch ho rahe hain.

---

## Task 4: Apne QR System Me Error Handling Audit Karo (25 min)

Apne QR System ke tool-calling code me jaake dekho:

1. Kya missing/invalid argument cases handle ho rahe hain, ya LLM directly execute ho jata hai bina validation ke?
2. Kya kahi raw error/exception LLM ko ya user ko directly expose ho raha hai?
3. Kya retry logic hai kahi (transient failures ke liye), aur kya wo sahi jagah use ho raha hai (sirf network/temporary errors pe, business logic failures pe nahi)?
4. Ek scenario socho jo abhi tumhara code handle nahi karta — usse kaise fix karoge (pseudocode likh sakte ho)?

**Deliverable:** Ek gap-analysis note — "Mere project me ye error scenarios handle nahi ho rahe abhi, production-ready banane ke liye ye add karna chahiye." Interview me directly useful — dikhata hai tumhe production concerns ki samajh hai, sirf happy-path coding nahi.

---

**Phase 2 Complete!** Ab tumne function calling — concept, schema design, chaining, aur error handling — sab cover kar liya hai. Agla phase — RAG + Vector DB — jaha LLM ko apne data se "ground" karna sikhoge.