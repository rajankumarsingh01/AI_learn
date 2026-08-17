# Phase 4 - Chapter 1: Practice Tasks - Ollama & Local LLM Basics

## Task 1: Installation & Basic Terminal Chat

Ollama install karo apne Windows system pe. `llama3.2:1b` model download karo (`ollama pull llama3.2:1b`). Terminal me `ollama run llama3.2:1b` se 3-4 questions poocho (Hinglish me bhi try karo) aur observe karo response quality aur speed kaisi hai apne hardware pe.

## Task 2: API Integration with Python (requests)

`query_ollama()` function use karke ek Python script likho jo 3 alag prompts ko local model ko bheje aur responses print kare. Note karo (comment me) — har response generate hone me kitna time laga (`time` module use karke measure karo).

## Task 3: Multi-Turn Chat with Context

`chat_with_ollama()` function use karke ek 4-message conversation banao jisme tum model ko pehle apna naam/context batao, phir baad me usse related question poocho — verify karo ki model context yaad rakh raha hai us conversation ke andar. Fir `ollama` Python library (`pip install ollama`) use karke same cheez implement karo aur dono approaches (raw requests vs library) ka code compare karo.

## Task 4: Local vs Cloud Comparison — Apne Projects Ke Liye Decision Framework

Apne teeno projects (QR System, Kaksha, API Observability) me se ek choose karo jaha LLM feature hai ya add karna chahte ho. Ek chhota analysis likho (markdown ya comments me):

- Us feature ke liye Local LLM (Ollama) use karna practical hoga ya nahi? Kyu?
- Same prompt ko Ollama (`llama3.2:1b`) aur agar possible ho to apne existing cloud LLM setup (jo Phase 1 me use kiya tha) dono se chalao, aur response quality compare karo (side-by-side, ek hi query ke liye)
- Socho aur likho: agar tumhare project ko "1000 users ek saath use kar rahe hain" scale pe le jaana ho, to local LLM approach kya problems face karega jo cloud LLM nahi karega? (Hint: concurrent requests, hardware bottleneck, consistency)