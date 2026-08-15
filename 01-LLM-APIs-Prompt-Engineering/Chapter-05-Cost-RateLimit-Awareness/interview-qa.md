# Phase 1 → Chapter 5: Cost & Rate-Limit Awareness — Interview Q&A

---

**Q1. LLM API cost kaise calculate hoti hai?**

Providers per-million-tokens basis pe charge karte hain, input aur output tokens alag rates pe (output usually zyada expensive hota hai). Total cost = (input tokens/1M × input rate) + (output tokens/1M × output rate). `usage` field response me actual token counts deta hai jisse cost calculate karte hain.

---

**Q2. Production me LLM cost optimize karne ke kya tareeke hain?**

Concise system prompts rakhna (repeated cost bachata hai), simple tasks ke liye cheaper/smaller model use karna, `max_tokens` sensibly set karna, response caching karna repeated queries ke liye, aur agar provider support kare toh batch API use karna.

---

**Q3. Rate limit kya hota hai aur kis form me exceed hone pe error milta hai?**

Rate limit ek fixed time window me allowed requests (RPM) ya tokens (TPM) ki limit hoti hai. Exceed hone pe API `429 Too Many Requests` HTTP status code ke saath error return karti hai.

---

**Q4. Exponential backoff kya hai aur ye important kyu hai?**

Ye ek retry strategy hai jisme har failed attempt ke baad wait time double hota jata hai (1s, 2s, 4s...) before retrying. Important hai kyunki agar server already rate-limited/overloaded hai, immediate/fixed-interval retries situation ko aur worse kar sakti hain — exponential wait server ko recover hone ka time deta hai.

---

**Q5. Production me LLM usage monitoring/logging kyu zaroori hai?**

Taaki daily/monthly cost track ho sake, unexpected usage spikes (bug ya abuse indicate karne wale) detect ho sakein, aur budget alerts set kiye ja sakein. Bina logging ke, ek buggy loop jo LLM ko repeatedly call kare, unnoticed reh sakta hai jab tak bill nahi aata.

---

**Q6. Kya har task ke liye sabse powerful/expensive model use karna chahiye? Kyu ya kyu nahi?**

Nahi. Simple tasks jaise classification ya basic extraction ke liye chhote/cheaper models kaafi accurate hote hain. Flagship model har jagah use karna unnecessary cost badhata hai bina proportional benefit ke — task complexity ke hisaab se model choose karna best practice hai.

---

**Q7. `max_tokens` ko "safe side" soch kar bahut high default set karna kyu problematic hai?**

Kyunki isse worst-case cost bhi high ho jata hai — agar model kabhi verbose ya repetitive output de de, poori `max_tokens` limit tak generate ho sakta hai, jisse cost aur latency dono badh jate hain bina zaroori benefit ke.

---

**Q8. Response headers me rate-limit information kaise milti hai, aur isse proactively kaise use kar sakte ho?**

Providers headers jaise `x-ratelimit-remaining-requests`, `x-ratelimit-remaining-tokens` bhejte hain har response ke saath. Production code me ye headers check karke tum proactively request rate slow kar sakte ho, bina 429 error aane ka wait kiye — isse smoother degradation hoti hai.

---

**Q9. Apne Observability/Monitoring System project se cost-tracking ka concept kaise connect karoge interview me?**

Bata sakte ho ki Monitoring System project me metrics collection, alerting, aur anomaly detection ka jo pattern seekha, wahi principle LLM cost-tracking me apply hota hai — har request ka usage log karna, thresholds set karna, aur anomalies (unexpected spikes) detect karke alert generate karna, essentially same observability mindset different domain me.

---

**Q10. Agar production app me achanak LLM cost spike ho jaye, debugging kaise approach karoge?**

Pehle logs check karoge — kaunse endpoint/feature se zyada requests aa rahi hain, kya koi loop accidentally repeated calls kar raha hai, kya `max_tokens` kahi unusually high set hai, ya kya koi user/bot abuse kar raha hai (rate limiting per-user missing ho sakta hai). Usage logs (Task 9 wala pattern) ye debugging fast karte hain kyunki per-request data already available hota hai.