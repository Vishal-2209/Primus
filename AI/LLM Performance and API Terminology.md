---
tags: [ai, llm, api, terminology]
created: 2026-08-15
status: reference
---

# LLM Performance and API Terminology

Ye ek reference/glossary note hai, kisi specific roadmap step ka part nahi hai. Jab bhi kisi LLM API ke docs padhte waqt in terms mein confuse ho, yahan wapas aa sakte ho.

## 1. Input

Input = jo information LLM ko di jaati hai.

Examples:
- User prompt
- System instructions
- Conversation history
- RAG documents
- Tool/API results
- Images or other supported inputs

Input ko internally tokens mein convert kiya jaata hai.

```text
User → Input → Tokenization → LLM
```

Example:

```text
Input:
"What is the weather in Ahmedabad?"
```

## 2. Output

Output = LLM dwara generate kiya gaya response.

```text
LLM → Output → User
```

Example:

```text
Input:
"What is the weather in Ahmedabad?"

Output:
"Ahmedabad is currently 31°C."
```

Output bhi tokens mein generate hota hai.

LLM APIs mein input aur output tokens ko separately measure kiya jaata hai because pricing often depends on them.

## 3. Context / Context Window

Context window = maximum number of tokens that an LLM ek request ke context mein process/consider kar sakta hai.

Context mein generally include ho sakta hai:

```text
System Prompt
+ User Prompt
+ Conversation History
+ RAG Context
+ Tool Results
+ Other Inputs
+ Output
```

Example:

```text
Model Context Window = 128K tokens

Total request context ≤ 128K tokens
```

### Important

Context Window is not equal to Memory.

Context window ka matlab ye nahi hai ki model permanently information remember karta hai.

- Context: current request/conversation mein available information
- Memory: information ko future interactions ke liye retain karna

### Example

Agar model ka context window 128K tokens hai aur tum 150K tokens ka content bhejte ho, to model us entire content ko ek saath process nahi kar sakta.

## 4. Latency

Latency = request bhejne ke baad response receive hone mein lagne wala time.

```text
Request
Down arrow
LLM Processing
Down arrow
Response
```

Agar response 3 seconds mein aata hai:

```text
Latency = 3 seconds
```

### TTFT (Time To First Token)

TTFT = request bhejne ke baad first output token receive hone mein lagne wala time.

```text
Request sent
1.2 sec
First token
```

Therefore:

```text
TTFT = 1.2 seconds
```

### Total Latency

Complete response receive hone tak ka total time.

```text
Request → First Token → Remaining Tokens → Complete Response
```

Streaming mein TTFT low hone se user ko response jaldi appear hota hai, even if complete response ko longer time lage.

## 5. Throughput

Throughput = system ek given amount of time mein kitna work/process kar sakta hai.

LLMs mein commonly:
- Tokens per second
- Tokens per minute
- Requests per second
- Requests per minute

### Example

Agar model 100 tokens/second generate karta hai aur response 1000 tokens ka hai, to approximately:

```text
1000 / 100 = 10 seconds
```

generation time lagega.

### Latency vs Throughput

| Term | Meaning |
|---|---|
| Latency | Ek request ko complete hone mein kitna time laga |
| Throughput | Ek unit time mein kitna work process hua |

Simple analogy:

```text
Latency: Ek car ko destination tak pahunchne mein kitna time
Throughput: Highway ek hour mein kitni cars handle kar sakti hai
```

## 6. Uptime

Uptime = service kitne percentage time available aur operational rahi.

Example:

```text
99.9% uptime
```

means service theoretically 0.1% time unavailable ho sakti hai.

Approximate monthly downtime:

```text
99 percent     : 7.2 hours
99.9 percent   : 43.2 minutes
99.99 percent  : 4.32 minutes
99.999 percent : 25.9 seconds
```

Higher uptime = higher reliability.

## Important LLM API Metrics

### RPM (Requests Per Minute)

Provider/API ek minute mein maximum kitni requests allow karta hai.

```text
RPM = Requests / Minute
```

Example:

```text
RPM = 60

Maximum ~60 requests/minute
```

### TPM (Tokens Per Minute)

Ek minute mein maximum kitne tokens process karne ki limit.

```text
TPM = Tokens / Minute
```

Example:

```text
TPM = 1,000,000

Maximum 1M tokens/minute
```

TPM mein input/output token accounting provider ke rules ke according differ kar sakti hai.

## Complete Picture

```text
                  LLM API
                     |
       ______________|______________
       |             |             |
     INPUT         CONTEXT       OUTPUT
       |           WINDOW          |
       |             |             |
       ______________|______________
                     |
                    LLM
                     |
              ________|________
              |               |
           LATENCY        THROUGHPUT
              |               |
          How fast?       How much?
                     |
                     |
                   UPTIME
               Is it available?
```

### One-line Definitions

```text
Input       : LLM ko kya diya?
Output      : LLM ne kya generate kiya?
Context     : LLM ek request mein kitna information handle kar sakta hai?
Latency     : Response aane mein kitna time laga?
Throughput  : Unit time mein kitna work/token process hua?
Uptime      : Service kitne time available rahi?
RPM         : Per minute kitni requests allowed?
TPM         : Per minute kitne tokens allowed?
TTFT        : First token aane mein kitna time laga?
```

Rule of thumb: Production LLM select karte waqt sirf intelligence/benchmark score mat dekho. Quality, Context Window, Latency, Throughput, Rate Limits, Cost, aur Uptime ko collectively evaluate karo.

## Related Notes
- [[00 Master Plan]]
- [[01 Machine Learning and LLM Fundamentals]]

## Tags
#ai #llm #api #tokens #latency #throughput
