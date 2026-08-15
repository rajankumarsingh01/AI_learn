# Phase 2 → Chapter 2: Tool Schema Design — Interview Q&A

---

**Q1. Achhi tool schema design production me kyu important hai?**

Kyunki weak schema design (ambiguous names, vague descriptions, missing types) se LLM galat tool choose kar sakta hai, ya sahi tool ke saath galat arguments extract kar sakta hai. Achhi schema design consistency aur accuracy dono improve karti hai — ye tutorial-level aur production-grade function calling ka main difference hai.

---

**Q2. Tool description likhne ka formula kya hai?**

Teen cheezein honi chahiye: (1) tool kya karta hai — core action, (2) kab use karna hai — trigger condition especially jab non-obvious ho, (3) kya return karta hai — agar disambiguation ke liye helpful ho. Isse LLM ko decision-making ke liye kaafi context milta hai.

---

**Q3. Enum parameters kyu use karne chahiye jaha applicable ho?**

Kyunki bina enum ke, LLM inconsistent string values generate kar sakta hai (jaise kabhi "Pending", kabhi "pending") jo backend validation fail kar sakti hain. Enum se LLM ko exact valid values pata hote hain, wo sirf unhi me se choose karega, consistency ensure hoti hai.

---

**Q4. Similar-sounding tools (jaise get_order_status aur get_menu_items) ke beech confusion kaise avoid karte hain?**

Explicit disambiguation description me likh kar — jaise "This is for EXISTING orders, NOT for browsing menu" jaisa pattern. "This is NOT for..." explicitly bolna model ko clear signal deta hai jab tools overlap-sounding hon.

---

**Q5. Required vs optional parameters kaise decide karte ho, aur optional parameter ka default value LLM ko kaise batate ho?**

`required` array me sirf wahi parameters daalte hain jo bina unke function meaningfully kaam nahi kar sakta. Optional parameters ke description me hi default value mention karte hain (jaise "quantity: defaults to 1 if not specified") taaki LLM samjhe missing case me kya assume karna hai.

---

**Q6. Nested objects/arrays wali tool schema kab zaroori hoti hai, example do.**

Jab tool ko complex/structured data chahiye ho — jaise `place_order` tool jisme multiple items (array of objects, har object me item_name aur quantity) aur delivery_address chahiye. Simple flat parameters se ye represent nahi ho sakta, isliye nested array-of-objects schema use karte hain.

---

**Q7. Generic parameter naam jaise `id` ya `value` kyu problematic hote hain?**

Kyunki naam se context clear nahi hota — agar multiple entities (order, user, item) involved hain, `id` field se pata nahi chalega kaunsa ID chahiye. Specific naam (`order_id`, `user_id`) aur description use karna zaroori hai clarity ke liye, especially jab tools me multiple similar-type parameters ho sakte hain.

---

**Q8. Production deploy karne se pehle tool schema ke kaunse edge cases test karne chahiye?**

Ambiguous input (jaise specific item na bole user), multiple valid interpretations wale complex messages, aur missing required information wale cases — check karna chahiye ki model gracefully handle karta hai (clarification maangta hai) ya galat assumptions bana leta hai.

---

**Q9. Sirf tool-level description likhna kaafi hai ya parameter-level description bhi zaroori hai?**

Parameter-level description bhi zaroori hai, especially jab parameter naam se meaning clear na ho. Sirf tool-level description se LLM ko overall context milta hai, lekin individual parameters ka exact meaning/constraints (jaise "not the user ID or item ID") parameter-level description se hi clear hota hai.

---

**Q10. Naming convention consistency LLM ke tool-selection accuracy ko kaise affect karti hai?**

LLM naam se bhi context leta hai — consistent verb-noun pattern (`get_X`, `add_X`, `update_X`) se model ko pattern recognize karna easy hota hai, especially jab system me 10-15+ tools ho. Inconsistent naming (mixed casing, unclear verb-noun order) se model confusion badh sakta hai large tool-sets me.