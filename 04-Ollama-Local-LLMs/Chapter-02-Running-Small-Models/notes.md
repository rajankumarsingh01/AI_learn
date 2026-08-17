# Phase 4 - Chapter 2: Customizing Ollama Models (Modelfile, Parameters, Streaming)

## Kya Hai Ye?

Chapter 1 me humne Ollama install kiya aur basic queries chalayi. Ab hum dekhenge ki model ka behavior kaise customize karte hain — system prompts set karna, generation parameters (temperature, top_p) tune karna, streaming responses handle karna, aur `Modelfile` use karke apna khud ka customized model variant banana.

Ye same concepts hain jo tumne Phase 1 (Prompt Engineering, Chapter 4-5) me cloud LLM APIs ke saath seekhe the — bas ab inhe local model ke context me apply kar rahe hain. Isse tumhe pata chalega ki ye concepts universal hain, chahe model cloud pe ho ya local.

## Kyu Zaroori Hai?

1. **Consistent Behavior Ke Liye Customization**: Bina system prompt/parameters set kiye, model generic responses degi. Agar tumhe specific persona ya format chahiye (jaise "hamesha Hinglish me answer do"), to customization zaroori hai.

2. **Performance Tuning Apne Hardware Ke Liye**: Kuch parameters (jaise `num_predict`, `num_ctx`) directly RAM/speed ko affect karte hain — tumhare 8GB RAM system pe ye settings correctly set karna practical difference banata hai.

3. **Streaming — User Experience Ka Critical Part**: Jaise ChatGPT/Claude ka response word-by-word aata hai (streaming), Ollama bhi ye support karta hai. Non-streaming me poora response wait karna padta hai — chhote local models pe ye especially slow feel hota hai, isliye streaming samajhna important hai.

4. **Modelfile — Custom Model Variants Banane Ka Tareeka**: Production use cases me kabhi-kabhi ek base model ko specific behavior ke saath "lock" karna hota hai (jaise ek fixed system prompt ke saath) — Modelfile ye karne deta hai, Docker ke Dockerfile jaisa concept.

5. **Interview Signal**: Ye dikhata hai ki tumne sirf basic API call nahi ki, balki model configuration aur production considerations (streaming, custom prompts) bhi samjhe hain — jo deeper understanding show karta hai.

## Kaise Kaam Karta Hai?

Ollama API generation ke time additional parameters accept karta hai jo model ke output behavior ko control karte hain, aur ek streaming mode support karta hai jisme response chunks me aata hai real-time.

---

## 1. System Prompt Set Karna

```python
import requests

def query_with_system_prompt(prompt, system_prompt, model="llama3.2:1b"):
    response = requests.post(
        "http://localhost:11434/api/generate",
        json={
            "model": model,
            "prompt": prompt,
            "system": system_prompt,
            "stream": False
        }
    )
    return response.json()["response"]

system = "Tum ek helpful assistant ho jo hamesha Hinglish me, concise answers deta hai. Kabhi English-only ya Hindi-only me answer mat do."

result = query_with_system_prompt("QR code kya hota hai", system)
print(result)
```

Ye Phase 1 Chapter 4 (Prompt Engineering Patterns) ke concepts ka direct application hai — system prompt model ka "persona" aur "rules" define karta hai.

## 2. Generation Parameters Tune Karna

```python
def query_with_params(prompt, model="llama3.2:1b", temperature=0.7, top_p=0.9, num_predict=200):
    response = requests.post(
        "http://localhost:11434/api/generate",
        json={
            "model": model,
            "prompt": prompt,
            "stream": False,
            "options": {
                "temperature": temperature,
                "top_p": top_p,
                "num_predict": num_predict,   # max tokens generate karne hain
                "num_ctx": 2048               # context window size
            }
        }
    )
    return response.json()["response"]
```

| Parameter | Kya Karta Hai | Rajan Ke Hardware Ke Liye Tip |
|---|---|---|
| `temperature` | Randomness control karta hai (0 = deterministic, 1+ = creative/random) | Factual tasks ke liye 0.2-0.3, creative ke liye 0.7-0.9 |
| `top_p` | Nucleus sampling — kitne probable tokens consider karne hain | Default 0.9 generally theek hai |
| `num_predict` | Max output tokens | Kam rakho (150-300) taaki response fast aaye chhote hardware pe |
| `num_ctx` | Context window size (kitna input+history model dekh sakta hai) | Chhota rakho (2048) — bada context window zyada RAM leta hai |

## 3. Streaming Response Handle Karna

```python
import requests
import json

def stream_ollama(prompt, model="llama3.2:1b"):
    response = requests.post(
        "http://localhost:11434/api/generate",
        json={"model": model, "prompt": prompt, "stream": True},
        stream=True
    )
    
    full_response = ""
    for line in response.iter_lines():
        if line:
            chunk = json.loads(line)
            token = chunk.get("response", "")
            print(token, end="", flush=True)  # real-time print, jaise ChatGPT
            full_response += token
            if chunk.get("done", False):
                break
    return full_response

stream_ollama("Ek chhoti kahani likho QR code ke baare me")
```

Streaming se user ko turant kuch dikhna shuru ho jaata hai (perceived latency kam hoti hai), chahe poora response generate hone me utna hi time lage jitna non-streaming me — ye UX ka important trick hai jo tum apne Node.js backend me bhi replicate kar sakte ho (Server-Sent Events ya WebSockets se).

## 4. Custom Modelfile Banana

Modelfile ek text file hai jisme tum base model, system prompt, aur parameters ko permanently ek naye custom model name ke saath "bake" kar sakte ho:

```dockerfile
# File: Modelfile (koi extension nahi, exact naam "Modelfile")
FROM llama3.2:1b

SYSTEM """
Tum QR Food Ordering System ka customer support assistant ho.
Hamesha Hinglish me, friendly tone me, concise answers do.
Agar order ya refund se related complex query ho, to bolo "Human support se connect kar raha hoon."
"""

PARAMETER temperature 0.4
PARAMETER num_predict 150
```

```bash
# Terminal me is Modelfile se custom model banao
ollama create qr-support-bot -f ./Modelfile

# Ab is custom model ko directly use kar sakte ho
ollama run qr-support-bot
```

```python
# Python se bhi use kar sakte ho
result = query_with_system_prompt("mera order cancel karna hai", "", model="qr-support-bot")
```

Ye approach production-style hai — ek baar Modelfile define karne ke baad, har baar system prompt repeat karne ki zaroorat nahi, model khud us behavior ke saath "baked" hai.

## 5. Model Ki Info Dekhna Aur Manage Karna

```bash
ollama list                    # saare downloaded models dekho
ollama show llama3.2:1b        # model details (parameters, template)
ollama rm <model_name>         # model delete karo (space free karne ke liye)
```

**Rajan ke liye tip**: Tumhare 8GB RAM/512GB SSD system pe, agar multiple models download kar liye aur space kam pad raha hai, to jo use nahi kar rahe unhe `ollama rm` se hata do — models 1-5GB tak le sakte hain each.

## Common Mistakes (Interview me pooche jate hain)

1. **`num_ctx` ko bahut bada rakhna** — Zyada context window size RAM usage significantly badhata hai. Chhote hardware pe agar `num_ctx` bahut bada rakha (jaise 8192+), to system slow ho sakta hai ya out-of-memory error aa sakta hai.

2. **Streaming ko galat parse karna** — Streaming response me har line ek JSON object hoti hai (poora response nahi), isse ek saath `json.loads()` poore response pe apply karna error dega. Line-by-line parse karna zaroori hai.

3. **Temperature 0 ko "always same output" samajhna** — Temperature 0 output ko highly deterministic banata hai, lekin bilkul 100% identical output guarantee nahi karta (kuch models me minor floating-point variations ho sakti hain). Interview me "temperature 0 = fully deterministic" bolna thoda oversimplified hai.

4. **Modelfile ko sirf system prompt store karne ka tareeka samajhna** — Modelfile parameters (temperature, num_predict) bhi bake kar sakta hai, sirf system prompt nahi. Isse ek complete "configured model" banta hai jo consistent behavior guarantee karta hai bina baar-baar code me parameters pass kiye.

5. **Har request pe naya custom model create karna** — `ollama create` ek one-time setup step hai (jaise Docker image build karna). Ise har API request pe repeat karna galat hai — ek baar create karo, phir bas usko `ollama run`/API se reuse karo.