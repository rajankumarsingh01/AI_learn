# Phase 4 - Chapter 2: Interview Q&A - Customizing Ollama Models

**Q1: Ollama me system prompt kaise set karte hain, aur ye kyu important hai?**
A: `system` parameter API request me pass karke, ya Modelfile me `SYSTEM` instruction se permanently bake karke. System prompt model ka persona, tone, aur behavioral rules define karta hai — bina isse, model generic/inconsistent responses degi. Ye Phase 1 ke Prompt Engineering concepts ka hi extension hai, bas local model context me applied.

**Q2: `temperature` aur `top_p` parameters kya control karte hain, aur inme kya farak hai?**
A: `temperature` randomness/creativity control karta hai — kam value (0.1-0.3) deterministic, factual responses deti hai, zyada value (0.7-1.0) creative, varied responses deti hai. `top_p` (nucleus sampling) ek different mechanism hai — ye sirf top probable tokens ka ek cumulative probability subset consider karta hai sampling ke liye. Dono milke output diversity control karte hain, aur practice me typically dono ko saath tune kiya jaata hai.

**Q3: Streaming response ko handle karte time kya technical consideration hai jo non-streaming me nahi hoti?**
A: Streaming response me data ek saath poora nahi aata — chunks (lines) me aata hai, jisme har line apna khud ka JSON object hoti hai indicating ek token/piece of response. Isse handle karne ke liye line-by-line iterate karna padta hai (`response.iter_lines()`), aur har chunk ko individually parse karke accumulate karna padta hai — poore response pe ek saath `json.loads()` apply nahi kar sakte jaise non-streaming me karte hain.

**Q4: Streaming ka user experience pe kya impact hota hai, given ki total generation time same rehta hai?**
A: Streaming "perceived latency" kam karta hai — user ko turant kuch dikhna shuru ho jaata hai (jaise ChatGPT ka word-by-word output), jisse wait karna kam frustrating lagta hai, chahe poora response complete hone me utna hi actual time lage jitna non-streaming me. Ye UX psychology ka principle hai — perceived speed, actual speed se kabhi zyada important hoti hai user satisfaction ke liye.

**Q5: Modelfile kya hai, aur ye kis real-world concept se similar hai?**
A: Modelfile ek configuration file hai jisme base model, system prompt, aur parameters ko ek naye custom model name ke saath permanently "bake" kiya jaata hai — `ollama create <name> -f Modelfile` se. Ye Docker ke Dockerfile concept se directly similar hai — jaise Dockerfile ek base image lekar usme customizations add karke naya image banata hai, Modelfile ek base LLM lekar usme behavior customizations add karke naya "model variant" banata hai.

**Q6: `num_ctx` parameter kya hai, aur resource-constrained hardware pe ise kaise handle karoge?**
A: `num_ctx` model ki context window size define karta hai — kitna input + conversation history model ek saath "dekh" sakta hai. Zyada `num_ctx` zyada RAM consume karta hai. Resource-constrained hardware (jaise 8GB RAM) pe ise chhota rakhna padta hai (jaise 2048 instead of 8192+), jo trade-off hai — kam context capacity, lekin system usable rehta hai bina crash/extreme-slowdown ke.

**Q7: Ek baar custom Modelfile-based model create karne ke baad, use production me kaise reuse karoge — kya har request pe recreate karna padta hai?**
A: Nahi, `ollama create` ek one-time setup operation hai (jaise Docker image ek baar build hota hai). Ek baar custom model (jaise `qr-support-bot`) create ho jaaye, use `ollama list` me dikhega aur baar-baar `ollama run qr-support-bot` ya API call se directly reuse kar sakte hain — recreate karna unnecessary aur wasteful hoga.

**Q8: Temperature 0 set karne ka matlab hai output 100% deterministic/identical hoga har baar — kya ye statement fully accurate hai?**
A: Largely accurate hai — temperature 0 sampling randomness ko almost eliminate kar deta hai, jisse model highly deterministic, greedy-decoding jaisa behavior deta hai. Lekin kuch edge cases me minor floating-point computation variations (hardware/implementation-level) se chhota output difference aa sakta hai. Interview me isse "highly deterministic" bolna zyada accurate hai "100% guaranteed identical" ki jagah.

**Q9: Agar tumhe apne QR System chatbot ko ek consistent "support agent" persona ke saath deploy karna ho local LLM pe, to kya approach loge?**
A: Modelfile approach use karunga — base model (`llama3.2:1b`) lekar, `SYSTEM` instruction me clear persona define karunga (tone, scope, escalation rules), aur appropriate `temperature` (low, jaise 0.3-0.4, kyunki support responses factual/consistent hone chahiye, creative nahi) set karunga. Isse `ollama create qr-support-bot -f Modelfile` se ek reusable custom model ban jaayega jo consistently us persona ke saath behave karega, bina har request me system prompt repeat kiye.

**Q10: Streaming implement karna Node.js/Express backend me (jo Rajan already use karta hai) kaise conceptually similar hoga?**
A: Node.js/Express me streaming typically Server-Sent Events (SSE) ya WebSockets se implement hoti hai — jaise Ollama chunks bhejta hai line-by-line, backend bhi frontend ko real-time chunks bhej sakta hai jaise LLM se data aata jaaye, bina poora response wait kiye. Concept same hai — client ko incremental data bhejna instead of ek single, complete response ka wait karana — chahe underlying transport mechanism (HTTP streaming, SSE, WebSocket) alag ho.