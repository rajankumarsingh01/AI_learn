# Phase 2 → Chapter 1: Function Calling Concept — Practice Tasks

---

## Task 1: Tool Definition Likhna Practice Karo (20 min)

Neeche diye gaye 3 scenarios ke liye tool definitions (JSON schema format, notes.md wale example jaisa) likho:

1. Ek tool jo user ka order history fetch kare (last N orders)
2. Ek tool jo restaurant ka current open/closed status check kare
3. Ek tool jo user ka delivery address update kare

Har tool me: `name`, clear `description`, aur proper `parameters` (with correct types) hone chahiye.

**Deliverable:** Teeno tool definitions JSON format me likho.

---

## Task 2: Real API Call — Function Calling Try Karo (30 min)

Anthropic API ka tool-use feature try karo:

```javascript
const response = await fetch("https://api.anthropic.com/v1/messages", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "x-api-key": process.env.ANTHROPIC_API_KEY,
    "anthropic-version": "2023-06-01"
  },
  body: JSON.stringify({
    model: "claude-sonnet-4-6",
    max_tokens: 300,
    tools: [
      {
        name: "get_order_status",
        description: "Get the current status and estimated delivery time of a food order by order ID",
        input_schema: {
          type: "object",
          properties: {
            order_id: { type: "integer", description: "The unique order ID" }
          },
          required: ["order_id"]
        }
      }
    ],
    messages: [
      { role: "user", content: "Mera order #4521 kaha hai?" }
    ]
  })
});

const data = await response.json();
console.log(JSON.stringify(data.content, null, 2));
```

Run karo aur dekho — model tool call karta hai ya nahi, aur kaunse arguments extract karta hai.

Ab isi tool ke saath ye query try karo: `"Paneer tikka me kya hota hai?"` — verify karo model tool call NAHI karta (kyunki ye general knowledge hai).

**Deliverable:** Dono outputs (tool call wala aur non-tool-call wala) paste karo, difference observe karo.

---

## Task 3: Poora Cycle Complete Karo — Fake Function Execute Karke (30 min)

Task 2 wale code ko extend karo taaki poora 5-step cycle complete ho:

```javascript
// Fake function jo actual database ki jagah hardcoded data return karega
function getOrderStatus(orderId) {
  return { status: "Out for delivery", eta: "15 minutes", order_id: orderId };
}

// Step 1: LLM se tool call request lo (Task 2 jaisa)
// Step 2: Check karo response me tool_use block hai
const toolUseBlock = data.content.find(block => block.type === "tool_use");

if (toolUseBlock) {
  // Step 3: Actually function execute karo
  const result = getOrderStatus(toolUseBlock.input.order_id);
  
  // Step 4: Result LLM ko wapas bhejo (naya request, tool_result ke saath)
  const followUpResponse = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: { /* same headers */ },
    body: JSON.stringify({
      model: "claude-sonnet-4-6",
      max_tokens: 300,
      tools: [ /* same tools */ ],
      messages: [
        { role: "user", content: "Mera order #4521 kaha hai?" },
        { role: "assistant", content: data.content }, // LLM ka tool call
        { 
          role: "user", 
          content: [
            {
              type: "tool_result",
              tool_use_id: toolUseBlock.id,
              content: JSON.stringify(result)
            }
          ]
        }
      ]
    })
  });
  
  const finalData = await followUpResponse.json();
  console.log(finalData.content[0].text); // Final natural language response
}
```

**Deliverable:** Poora console output — dikhao ki LLM ne kaise final natural-language response banaya fake database result se.

---

## Task 4: Apne QR System Ka Tool Calling Code Audit Karo (25 min)

Apne actual QR Food Ordering System project me jao:

1. Kaunse tools defined hain? List banao (name + description).
2. Kya har tool ki description clear/specific hai, ya kuch vague hain jo improve ho sakti hain?
3. Function execution kaha ho raha hai (kaunsi file/module)?
4. Parallel tool calls kahi handle ho rahe hain, ya sab sequential hai?

**Deliverable:** Ek audit note — apne actual tools ki list, aur agar koi description improve karne layak lagi, naya version likho. Ye interview me directly kaam aayega: "Mere project me maine ye tools define kiye the aur unka purpose ye tha..."