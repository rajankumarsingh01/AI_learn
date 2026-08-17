# Phase 4 - Chapter 2: Practice Tasks - Customizing Ollama Models

## Task 1: System Prompt Experimentation

`query_with_system_prompt()` function use karke same question ("mujhe ek achha laptop suggest karo") ko 3 alag system prompts ke saath poocho — (1) "Tum ek technical expert ho, detailed specs do", (2) "Tum ek dost ho, casual advice do", (3) "Tum sirf ek line me answer dete ho". Compare karo teeno responses aur note karo (comment me) — system prompt se output kitna change hota hai.

## Task 2: Parameter Tuning Experiment

`query_with_params()` function use karke same creative prompt ("ek chhoti kahani likho robot ke baare me") ko `temperature=0.1` aur `temperature=0.9` dono ke saath 2-2 baar chalao (total 4 responses). Compare karo — kya low temperature responses zyada similar/predictable hain compare to high temperature responses? Apna observation likho.

## Task 3: Implement Streaming

`stream_ollama()` function use karke ek lambi response wale prompt (jaise "5 tips do healthy lifestyle ke liye, detail me") ko stream mode me chalao. Verify karo ki tokens real-time print ho rahe hain (na ki ek saath end me). `time` module use karke measure karo — pehla token aane me kitna time laga (time-to-first-token), aur poora response complete hone me kitna time laga — dono numbers note karo.

## Task 4: Apne QR System Ke Liye Custom Modelfile Banao

Apne QR Food Ordering System ke liye ek custom Ollama model banao:

- Ek Modelfile likho jisme base model `llama3.2:1b` ho, system prompt QR System ke customer-support-assistant persona ke liye customized ho (naam, scope: menu/order/refund queries, tone: Hinglish-friendly), aur appropriate `temperature`/`num_predict` parameters set hon
- `ollama create qr-support-bot -f Modelfile` se model create karo
- Is naye model ko 3 different test queries ke saath test karo (ek menu-related, ek refund-related, ek out-of-scope jaise "aaj mausam kaisa hai" — verify karo model politely bolta hai ki ye uske scope me nahi hai agar tumne system prompt me aisa specify kiya ho)
- Socho aur likho (comment ke form me): agar tum is custom local model ko apne actual Node.js/Express QR System backend me integrate karna chaho (Ollama API ko backend se call karke), to production me kya concerns honge — specifically jab multiple users ek saath request bhejein tumhare single local machine ko?