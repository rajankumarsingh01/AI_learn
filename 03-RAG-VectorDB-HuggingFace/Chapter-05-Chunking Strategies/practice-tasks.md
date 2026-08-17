# Phase 3 - Chapter 5: Practice Tasks - Chunking Strategies

## Task 1: Fixed-Size Chunking with Overlap

Ek lamba paragraph (kam se kam 1000 characters — kisi article/blog se copy kar sakte ho) lo, aur `fixed_size_chunk()` function use karke ise `chunk_size=200, overlap=30` ke saath chunks me todo. Print karo total chunks aur pehle 2 chunks ka content — dekho overlap kaise dikhta hai practically.

## Task 2: Sentence-Based vs Fixed-Size Comparison

Same paragraph ko `sentence_chunk()` (max_sentences=3) se bhi chunk karo. Dono approaches (Task 1 aur Task 2) ke chunks ko compare karo — kaunsa approach zyada "readable"/semantically complete chunks banata hai? Comment me apna observation likho with specific examples (jaise "Task 1 me ye sentence beech me kat gaya, Task 2 me nahi").

## Task 3: Recursive Splitting Implementation

`recursive_split()` function ko ek 3-paragraph document pe test karo (paragraphs `\n\n` se separated hon). `chunk_size=300` rakho. Verify karo ki chunks paragraph boundaries respect kar rahe hain jab tak possible ho. Fir chunk_size ko 100 kar ke dekho — kya ab chunks paragraph ke andar bhi split ho rahe hain (sentence level pe)?

## Task 4: Apne QR System/Kaksha Me Apply Karo

Socho tumhare QR System ya Kaksha me ek lamba document hai jo RAG me daalna hai — jaise QR System ki "Complete Menu + Policies" document, ya Kaksha ka "Course Syllabus + FAQ" document (5-10 paragraphs, dummy content chalega).

- Ye document banao (dummy text, but realistic structure — headers ke saath, jaise "## Refund Policy", "## Delivery Time")
- `chunk_with_metadata()` function use karke ise chunk karo, jisme har chunk ka metadata ho: `source`, `chunk_index`, aur ek naya field `section` (jo tum manually ya heading-detection se determine karo)
- Chapter 3 ke ChromaDB collection me in chunks ko store karo (with metadata)
- Ek query chalao jisme metadata filter ho (jaise `where={"section": "refund_policy"}`) aur verify karo ki sirf us section ke chunks retrieve ho rahe hain
- Socho aur likho (comment ke form me): agar chunk_size bahut chhota rakha (jaise 50 characters) to kya problem aayegi retrieval me, aur agar bahut bada rakha (jaise 2000 characters) to kya problem aayegi — apne QR System ke real example se explain karo