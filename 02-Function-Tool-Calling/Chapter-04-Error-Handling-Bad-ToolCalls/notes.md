# Phase 2 → Chapter 4: Error Handling — Bad Tool Calls

## Kya/Kyu/Kaise

**Kya hai ye?**
Ab tak humne "happy path" dekha — LLM sahi tool choose karta hai, sahi arguments deta hai, function successfully execute hota hai. Real production me aisa hamesha nahi hota. LLM kabhi galat tool choose kar sakta hai, kabhi galat/missing arguments de sakta hai, ya kabhi function khud fail ho sakta hai (database down, network error). Ye chapter sikhata hai in sab failure modes ko gracefully handle karna.

**Kyu zaroori hai?**
Ye phase 2 ka sabse "production-readiness" wala chapter hai. Fresher developers zyadatar sirf happy-path code likhte hain — demo me achha chalta hai, lekin real users ke random/unexpected inputs se crash ho jata hai. Interview me senior engineers exactly yehi test karte hain: "agar ye fail ho jaye toh kya hoga?"

**Kaise handle karte hain?**
Teen tarah ki failures hoti hain: (1) LLM ne galat tool choose kiya, (2) LLM ne arguments galat/incomplete diye, (3) Tool khud execute karte waqt fail ho gaya (external system error). Har ek ka alag handling approach hai.

---

## 1. Type 1: LLM Ne Galat Tool Choose Kiya

Ye rare hota hai agar schema design achhi hai (Chapter 2 yaad karo), lekin ho sakta hai — especially ambiguous queries pe.

```
User: "Mera order dikhao" (ambiguous — "order" ka matlab "menu se order karna" ya "existing order dekhna"?)

Agar LLM galat "place_order" tool choose kar le jab user actually
"get_order_status" chahta tha...
```

**Prevention (best fix):** Schema descriptions improve karo (Chapter 2) — ye root cause fix hai, downstream error handling sirf safety-net hai.

**Detection:** Agar tool call ke arguments "empty" ya nonsensical lag rahe hain user ke actual message se (jaise user ne koi specific order mention nahi kiya but LLM ne `place_order` call kar diya bina items ke), ye red flag hai.

---

## 2. Type 2: Missing Ya Invalid Arguments

Ye sabse common failure type hai. LLM ne sahi tool choose kiya, lekin arguments incomplete/galat hain.

### Example:
```javascript
// User: "Mera order cancel kar do" (order_id nahi bataya)

// LLM tool call:
{
  name: "cancel_order",
  input: { order_id: null }  // ya field hi missing hai
}
```

**Handling pattern — Backend validation:**
```javascript
function handleToolCall(toolName, toolInput) {
  if (toolName === "cancel_order") {
    if (!toolInput.order_id) {
      // Function execute mat karo, error result wapas bhejo LLM ko
      return {
        success: false,
        error: "order_id is missing. Please ask the user which order they want to cancel."
      };
    }
    // ... actual cancellation logic
  }
}
```

**Key insight:** Jab LLM ko ye error result milta hai, wo **khud user se follow-up sawal poochega** ("Kaunsa order cancel karna hai? Order ID bata dijiye") — is tarah conversational recovery ho jati hai, crash nahi hota.

---

## 3. Type 3: Tool Execution Khud Fail Ho Gaya (External Errors)

Database down ho sakta hai, network timeout ho sakta hai, third-party API fail ho sakti hai.

```javascript
async function handleToolCall(toolName, toolInput) {
  if (toolName === "cancel_order") {
    try {
      const result = await database.cancelOrder(toolInput.order_id);
      return { success: true, data: result };
    } catch (error) {
      // Database error — LLM ko friendly error message do, raw stack trace NAHI
      return {
        success: false,
        error: "Unable to process the cancellation right now due to a system issue. Please try again in a moment."
      };
    }
  }
}
```

**Critical practice:** Kabhi bhi raw error/stack trace LLM ko mat bhejo (security risk — internal system details leak ho sakti hain, aur LLM confusing/technical response bana sakta hai user ko). Hamesha ek **clean, user-friendly error message** wrap karke bhejo.

---

## 4. Retry Logic — Kab Retry Karna Chahiye, Kab Nahi

| Error Type | Retry Karna Chahiye? |
|---|---|
| Network timeout, temporary server error (5xx) | Haan — exponential backoff ke saath (Phase 1 Chapter 5 yaad karo) |
| Missing/invalid arguments (LLM ki galti) | Nahi — retry se same result aayega, LLM ko clarification maangne do |
| Business logic failure (jaise "order already delivered, cannot cancel") | Nahi — ye permanent failure hai is order ke liye, retry se kuch nahi badlega |
| Rate limit hit (429) | Haan — backoff ke saath |

---

## 5. Validation Before Execution — Defense Layer

Best practice hai ki tool ke arguments ko **execute karne se pehle validate** karo, sirf LLM pe trust mat karo ki wo hamesha sahi data dega:

```javascript
function validateCancelOrderInput(input) {
  const errors = [];
  
  if (!input.order_id || typeof input.order_id !== "number") {
    errors.push("order_id must be a valid number");
  }
  
  if (errors.length > 0) {
    return { valid: false, errors };
  }
  
  return { valid: true };
}

async function handleToolCall(toolName, toolInput) {
  if (toolName === "cancel_order") {
    const validation = validateCancelOrderInput(toolInput);
    if (!validation.valid) {
      return { success: false, error: validation.errors.join(", ") };
    }
    // proceed with execution
  }
}
```

**Kyu ye zaroori hai:** LLM ka schema-following usually achha hota hai, lekin 100% guaranteed nahi (especially bina `strict` mode ke, ya edge cases me). Backend validation ek safety net hai jo bugs/security issues prevent karti hai.

---

## 6. QR System Connection — Real Scenarios

Tumhare QR System me ye errors ho sakte hain:

```
Scenario 1: User bolta hai "wo cheez add karo" (specific item name nahi diya)
→ add_item_to_cart tool call with item_name: "" ya unclear
→ Validation catch kare, LLM ko clarification maangne do

Scenario 2: Restaurant se connection fail ho jaye order place karte waqt
→ place_order function try-catch me wrapped ho, friendly error return kare
→ "Abhi order place nahi ho pa raha, thodi der me try karein" jaisa response

Scenario 3: Item out of stock nikle jab add karne ki koshish ho
→ Ye business logic failure hai (permanent for this attempt)
→ LLM ko clear signal do: "item unavailable", LLM user ko alternative suggest kar sakta hai
```

---

## Common Mistakes (Interview me pooche jate hain)

1. **Raw error/stack trace LLM ko bhej dena** — security risk hai, aur LLM confusing response bana sakta hai. Hamesha clean error message wrap karo.
2. **Har error pe retry karna, chahe LLM ki galti ho ya business logic failure** — sirf transient/network errors pe retry karo, permanent failures pe LLM ko clarification maangne do.
3. **Backend validation skip karna, sirf LLM schema pe trust karna** — schema following guaranteed nahi hoti hamesha, validation ek zaroori safety layer hai.
4. **Error result ko silently ignore karna** — agar tool call fail hua aur ye result LLM ko explicitly nahi bataya gaya (success: false wagera), LLM assume kar sakta hai sab theek hua, aur galat final response bana sakta hai user ko.