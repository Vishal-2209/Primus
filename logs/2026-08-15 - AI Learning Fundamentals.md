---
created: 2026-08-15
status: complete
---

# AI Learning Fundamentals

This is a complete log of everything learned about AI, LLMs, and agent building on 15 Aug 2026. Covers ML basics through to practical agent frameworks.

---

## Machine Learning Basics

Machine Learning gives models the ability to learn patterns from data and make predictions. When people say "the model thinks," what's actually happening is statistical pattern matching. Neural networks, decision trees, and other models all learn patterns from training data and use those patterns to predict or decide. There's no literal "thinking" or "understanding" happening inside.

## How LLMs Work

LLMs (Large Language Models) are part of the ML family. Their specific job is predicting the next token (word or sub-word) given some input. The process repeats until a full response is generated.

This means LLMs have no literal "understanding" or "consciousness." They're just very powerful next-token predictors trained on massive amounts of data, which makes their output look human-like and coherent.

## Why NLP Matters

LLMs need to communicate in natural language, and logic/patterns alone aren't enough. NLP (Natural Language Processing) concepts are needed so that:

- Text can be represented numerically for the model to process
- Language structure (grammar, context, meaning) can be captured to some extent
- Model output can be converted back into readable, coherent language

This is why tokenization is important.

## Tokenization

Tokenization splits raw text into small units called tokens. These can be words, sub-words, or single characters. The model can't work with text directly, so after tokenization each token is converted to a number (token ID) that the model processes internally.

```text
Text: "What is the weather?"
Tokens (approx): ["What", " is", " the", " weather", "?"]
Token IDs: [1023, 318, 262, 6193, 30]
```

This is why LLM APIs measure input/output in tokens, and pricing is based on token counts. Related metrics (context window, input/output tokens, TPM, etc.) are covered in the API Terminology section below.

## How a Prompt Actually Executes

When you give a prompt to an LLM and get a response, here's what happens behind the scenes:

```text
User Prompt (text)
        ↓
Tokenization (text → token IDs)
        ↓
Model processes tokens through its layers
        ↓
Model predicts next token, one by one, repeatedly
        ↓
Generated tokens are converted back to text (detokenization)
        ↓
Final Output (text) is returned to the user
```

Even though this process is fast, deep down it's a repetitive next-token-prediction loop. This is why ambiguous or poorly structured prompts lead to ambiguous outputs — the model only has pattern-based prediction, no real "understanding."

---

## LLM Providers

LLM providers are companies that let you access their trained models through APIs. Each provider issues an API key, and you use that key to call their models.

### Major Providers

- OpenAI (GPT series)
- Anthropic (Claude series)
- Google (Gemini series)
- Meta (Llama series)
- Mistral AI
- Cohere
- xAI (Grok series)
- DeepSeek
- Alibaba (Qwen series)
- NVIDIA (Nemotron series)
- Amazon (Titan, Bedrock for multiple models)
- Microsoft Azure (Azure OpenAI Service)
- Together AI, Groq (infrastructure providers serving open models with fast inference)

This list isn't complete, but these are the most commonly used names right now.

### The Problem with Multiple Providers

If you want to try models from different providers (one task with OpenAI, another with Anthropic, another with an open-source model), traditionally you need:

- A separate account for each provider
- A separate API key for each
- Slightly different request/response formats to handle (until they're OpenAI-compatible)

This creates friction when experimenting or comparing models.

### Solution: Aggregator Platforms

Aggregator platforms solve this by letting you access many providers' models through a single API key and mostly consistent request format. The most well-known is OpenRouter.

### Why OpenRouter

I'm using OpenRouter for this entire learning journey because:

- One API key gives access to hundreds of models from different providers
- Request/response format is mostly OpenAI-compatible, which is already familiar
- Free tier models are available for experimenting
- If one provider goes down or hits rate limits, fallback options are available

OpenRouter access works through three levels of abstraction, covered next.

---

## OpenRouter: Levels of Abstraction

OpenRouter sets up three levels of access. Each level trades off control vs convenience. More control means writing more code yourself; more convenience means more implementation details are hidden.

```text
Level 1: No SDK          -> Most control, most code
Level 2: Client SDK      -> Medium abstraction
Level 3: Agents SDK      -> Most convenience, least control
```

### Level 1: No SDK (Raw Requests)

No SDK is used here. You make HTTP calls directly to OpenRouter's REST endpoint using the `requests` library. If you don't use an SDK, there are no SDK-related problems, but the code will be longer.

The benefit is that every part of the process is clear — you write the manual loop yourself and inspect every field of the request/response.

```python
import requests
import json
import os

response = requests.post(
    url="https://openrouter.ai/api/v1/chat/completions",
    headers={
        "Authorization": "Bearer " + os.getenv("OPENROUTER_API_KEY")
    },
    data=json.dumps({
        "messages": [
            {
                "role": "user",
                "content": "Hi, how are you?"
            }
        ],
        "models": ["cohere/north-mini-code:free", "nvidia/nemotron-3.5-lightning:free"],
    })
)

data = response.json()
print(data["choices"][0]["message"]["content"])
```

Here `models` is passed as an array instead of a single `model` string. This gives OpenRouter a fallback list — if the first model isn't available or fails, the next one is tried.

#### Understanding the Response Structure

The raw JSON response looks like this:

```json
{
  "id": "gen-1786776316-31WSQKcyLVIbJXaZkHb1",
  "object": "chat.completion",
  "model": "nvidia/nemotron-3.5-lightning:free",
  "provider": "Nvidia",
  "choices": [
    {
      "index": 0,
      "finish_reason": "stop",
      "message": {
        "role": "assistant",
        "content": "Hello! I'm doing well, thank you for asking. How can I help you today?",
        "reasoning": "..."
      }
    }
  ],
  "usage": {
    "prompt_tokens": 22,
    "completion_tokens": 206,
    "total_tokens": 228,
    "cost": 0,
    "completion_tokens_details": {
      "reasoning_tokens": 179
    }
  }
}
```

Important fields:

- `choices[0].message.content` — the actual final answer to show the user
- `choices[0].message.reasoning` — if the model is reasoning-capable (like Nemotron), its internal thinking process appears here, separate from the final content
- `usage` — breakdown of input/output tokens; if reasoning was used, its tokens are counted separately in `completion_tokens_details.reasoning_tokens`
- `provider` — which underlying provider (e.g., Nvidia) the request was actually routed to

All these fields are directly available in the raw response because we parsed the JSON ourselves. With a Client SDK, the same data is accessed through object-style access, but the underlying structure stays the same.

#### Important Security Note

Never hardcode API keys directly in code or share them with anyone. Always keep them in a `.env` file or environment variable:

```python
import os
api_key = os.getenv("OPENROUTER_API_KEY")
```

### Level 2: Client SDK

A Client SDK (like the `openrouter` Python package, or using the OpenAI SDK with a changed `base_url` as a drop-in replacement) adds a layer on top so you don't have to build the full JSON manually every time.

Under the hood, the SDK still makes the same HTTP request you wrote in Level 1. The complexity is just hidden inside a function call.

The first version I built had only one round: the LLM returns a tool_call, it gets executed, and then it stops. That didn't give a final natural language answer. The proper, spec-compliant version puts this in a loop where the tool result is fed back into the `messages` list with `role: "tool"`, and the LLM is called again until it gives final content.

```python
from openrouter import OpenRouter
import os
import json
from dotenv import load_dotenv
import tools_for_sdks

load_dotenv()

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the weather of a city",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {"type": "string", "description": "City name"}
                },
                "required": ["city"]
            }
        }
    }
]

with OpenRouter(api_key=os.getenv("OPENROUTER_API_KEY")) as client:
    messages = [
        {
            "role": "user",
            "content": input("Enter task : ")
        }
    ]

    while True:
        response = client.chat.send(
            model="openrouter/free",
            messages=messages,
            tools=tools
        )

        ai_message = response.choices[0].message
        tool_calls = ai_message.tool_calls

        if tool_calls:
            messages.append(ai_message)
            for call in tool_calls:
                if call.function.name == "get_weather":
                    args = json.loads(call.function.arguments)
                    result = tools_for_sdks.get_weather(args["city"])
                    messages.append({
                        "role": "tool",
                        "tool_call_id": call.id,
                        "content": json.dumps(result)
                    })
        else:
            print(ai_message.content)
            break
```

The actual tool execution happens in a separate file (`tools_for_sdks.py`), where each tool hits a real API (like `get_weather` calling OpenWeatherMap). The LLM only decides which tool to use and what arguments to pass. The actual work (hitting the API) is always done by our code — exactly the principle from the Agents section.

#### What's Happening Inside the Loop, Step by Step

- `messages` is defined outside the loop as a list and updated in each iteration, preserving full conversation history
- When the LLM returns `tool_calls`, the **LLM's own message** (`ai_message`) is added to `messages` first. This is required — per spec, a `role: "tool"` message without a preceding assistant tool_call is invalid conversation state
- Each tool call's result is added with `role: "tool"` along with `tool_call_id`, so if multiple tools were called at once, the LLM knows which result belongs to which call
- The loop keeps running until the LLM stops requesting tools. As soon as it gives direct `content` (meaning it doesn't need any more tools), the `else` branch triggers, the final answer is printed, and `break` terminates the loop

This entire structure is the "agentic behavior" described in the Agents section. Frameworks like CrewAI automate this loop internally.

#### Client SDK Isn't Limited to Chat

Client SDKs aren't just for chat completions. For example, the OpenAI SDK can be used with OpenRouter for audio transcription by changing the `base_url`:

```python
from openai import OpenAI
import os
from dotenv import load_dotenv

load_dotenv()
client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key=os.getenv("OPENROUTER_API_KEY"),
)

with open("audio_file.mp3", "rb") as f:
    result = client.audio.transcriptions.create(
        model="openai/whisper-large-v3",
        file=f,
    )

print(result.text)
```

This shows that Client SDK is a general-purpose layer that follows a consistent pattern across chat and other AI services (transcription, embeddings, image generation, etc.).

#### Core Idea of Client SDK

Less code means fewer chances of errors. So Client SDK works the traditional way through RESTful APIs but makes the code shorter and implementation easier. The underlying request is the same one you'd write manually in Level 1.

### Level 3: Agents SDK

Agents SDK (like OpenRouter's `@openrouter/agent` package) provides the highest abstraction. It handles the entire multi-turn loop (LLM call, tool_call detection, execution, result feeding, re-calling) internally through a single function call (like `callModel`).

One limitation right now: it's only supported in TypeScript/JavaScript, not Python. So I haven't tried it yet, since I've been using Python-based Client SDK and No SDK approaches so far.

### Comparison of All Three Levels

| Level | Control | Code Amount | Language | Status |
|---|---|---|---|---|
| No SDK | Most | Most | Any (raw HTTP) | Tried |
| Client SDK | Medium | Medium | Python, JS, wherever SDK exists | Tried |
| Agents SDK | Least | Least | TypeScript/JavaScript only | Not tried yet |

---

## Agents and Workflows

### The Real Topic

After understanding how LLMs work (word prediction, tokenization, prompt execution), the real question is: how can we use LLMs to make things practically better? The answer is by designing agents and workflows.

### Designing vs Building Are Different Tasks

Designing an agent or workflow (planning what steps there will be, which tools are needed, where decision points are) is one thing. Actually implementing it is another.

Implementation can be done with traditional coding and DSA (data structures and algorithms) — fully manual. But in practice, people use "agent frameworks" so that:

- Repetitive boilerplate code doesn't need to be written again and again
- Common patterns (loop, tool execution, state management) are already ready
- Implementation becomes easier and more standardized

Frameworks (like LangGraph, CrewAI) are covered in the hands-on roadmap.

### What an Agent Actually Is

This is the most important mental model to get clear: **an agent isn't some separate, magical entity. An agent is basically just an LLM** that has been given some tools available to it, and when needed, it generates the required JSON schema to use those tools (meaning the tool's name and its parameters, in structured format).

```text
Agent = LLM + Tool Definitions (JSON schemas)
```

When we say "the agent decided to check the weather," what actually happened is the LLM generated a JSON object like:

```json
{
  "tool": "get_weather",
  "arguments": { "city": "Ahmedabad" }
}
```

The LLM doesn't execute the tool itself. It only decides which tool is needed and what arguments to pass. Actual execution always happens outside, in our own code.

### Where Does "Agentic" Behavior Come From

If the LLM only generates a schema, how does the full multi-step, tool-using behavior get built? The answer is: a loop.

```text
Call LLM (with tools available)
        ↓
Check: did LLM return a tool_call or a direct answer?
        ↓
If tool_call:
    Actually execute the tool (our code)
    Feed the result back into the conversation
    Call LLM again
        ↓
This loop keeps running until a final answer is received
```

This is the same manual loop you write with raw SDK, and this is the same loop that agent frameworks (LangGraph, CrewAI, or OpenRouter's Agents SDK) automate internally. The benefit of using a framework is that this loop, state management, and error handling are handled automatically, but the underlying concept is exactly the same as what you'd write manually.

### Summary

- LLMs only do prediction and schema generation
- Execution is always done by external code (ours, or the framework's internal code)
- "Agent" is a label for the complete setup: LLM + tools + loop
- Frameworks make this setup easier to implement, but the concept isn't new

---

## Tools, MCP, and Skills

These three concepts look different, but behind them is one common principle: **the LLM only generates a schema or decision, actual execution always happens outside, in code.** Understanding this is the most important thing, because all three are based on this principle.

### Tools

A tool is basically a function description given to the LLM in JSON schema format. It contains:

- Tool name
- Description (tells the LLM what this tool does)
- Parameters (which arguments are needed, what types)

```json
{
  "type": "function",
  "function": {
    "name": "get_weather",
    "description": "Get the weather of a city",
    "parameters": {
      "type": "object",
      "properties": {
        "city": { "type": "string", "description": "City name" }
      },
      "required": ["city"]
    }
  }
}
```

When the LLM feels that this tool is needed to answer the user's question, it doesn't call the tool itself. The LLM only outputs this:

```json
{
  "name": "get_weather",
  "arguments": { "city": "Ahmedabad" }
}
```

After this, our own code parses this JSON, calls the actual function (which hits a real API like OpenWeatherMap), and feeds the result back to the LLM if a final natural language answer is needed.

### MCP (Model Context Protocol)

If tool schemas and execution logic need to be written separately for every project, it becomes repetitive, and with different frameworks/clients having their own tool-integration formats, things get even more inconsistent.

MCP is a standard protocol that solves this problem. Tool definitions and execution logic can be packaged once as a "server," and then that server can be plugged into any MCP-compatible client (Claude Desktop, or any other MCP client) without rewriting the schema and logic.

Analogy: just like USB-C has become a common standard for all devices, MCP is a common standard for tool integration.

```text
Without MCP: Each client defines its own tool format
With MCP: Build one MCP server, plug it into any MCP client
```

The principle stays the same here too: inside the MCP server, the LLM only decides which tool is needed, actual execution happens in the MCP server's code, not inside the LLM.

### Skills

Skills is a relatively new pattern (like Claude Skills) where instead of putting all instructions into one big system prompt, they're organized into small, modular instruction files. The LLM only knows a short description of these files upfront, and when a specific task seems relevant, it dynamically loads the full file.

This pattern is called "progressive disclosure" — instead of putting everything into context upfront, load it only when needed.

```text
Without Skills: One giant system prompt, everything always in context
With Skills: Small modular files, only relevant one loads when needed
```

The same principle applies here. The LLM doesn't run the file-loading mechanism itself — it only decides "this skill name seems relevant to me," and the rest of the loading, file reading, and actual use is handled by the surrounding code/system.

### Common Thread

In Tools, MCP, and Skills, one thing repeats:

```text
LLM's job: decide and generate schema
Code's job: actually execute that decision
```

This mental model will be useful again and again throughout the agent-building process — whether writing a manual loop with raw SDK, building a graph in LangGraph, or designing an MCP server. The LLM never "executes" anything itself; it only gives structured decisions.

---

## LLM Performance and API Terminology

This is a reference/glossary section. When you get confused by terms in LLM API docs, come back here.

### Input

Input = the information given to the LLM.

Examples:

- User prompt
- System instructions
- Conversation history
- RAG documents
- Tool/API results
- Images or other supported inputs

Input is internally converted to tokens.

```text
User → Input → Tokenization → LLM
```

### Output

Output = the response generated by the LLM.

```text
LLM → Output → User
```

Output is also generated in tokens. LLM APIs measure input and output tokens separately because pricing often depends on them.

### Context / Context Window

Context window = the maximum number of tokens an LLM can process/consider in a single request's context.

Context generally includes:

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

#### Important

Context Window is not equal to Memory.

Having a context window doesn't mean the model permanently remembers information.

- Context: information available in the current request/conversation
- Memory: retaining information for future interactions

#### Example

If the model's context window is 128K tokens and you send 150K tokens of content, the model can't process all of it at once.

### Latency

Latency = the time taken to receive a response after sending a request.

```text
Request
    ↓
LLM Processing
    ↓
Response
```

If the response comes in 3 seconds:

```text
Latency = 3 seconds
```

#### TTFT (Time To First Token)

TTFT = the time taken to receive the first output token after sending a request.

```text
Request sent
1.2 sec
First token
```

Therefore:

```text
TTFT = 1.2 seconds
```

#### Total Latency

Total time until the complete response is received.

```text
Request → First Token → Remaining Tokens → Complete Response
```

In streaming, low TTFT makes the response appear quickly to the user, even if the complete response takes longer.

### Throughput

Throughput = how much work/process a system can handle in a given amount of time.

With LLMs, commonly measured as:

- Tokens per second
- Tokens per minute
- Requests per second
- Requests per minute

#### Example

If the model generates 100 tokens/second and the response is 1000 tokens, approximately:

```text
1000 / 100 = 10 seconds
```

generation time will be needed.

#### Latency vs Throughput

| Term | Meaning |
|---|---|
| Latency | How much time one request took to complete |
| Throughput | How much work/tokens were processed in a unit of time |

Simple analogy:

```text
Latency: How much time one car takes to reach its destination
Throughput: How many cars a highway can handle in one hour
```

### Uptime

Uptime = the percentage of time a service was available and operational.

Example:

```text
99.9% uptime
```

means the service is theoretically unavailable 0.1% of the time.

Approximate monthly downtime:

```text
99 percent     : 7.2 hours
99.9 percent   : 43.2 minutes
99.99 percent  : 4.32 minutes
99.999 percent : 25.9 seconds
```

Higher uptime = higher reliability.

### Important LLM API Metrics

#### RPM (Requests Per Minute)

The maximum number of requests the provider/API allows per minute.

```text
RPM = Requests / Minute
```

#### TPM (Tokens Per Minute)

The maximum number of tokens allowed to be processed per minute.

```text
TPM = Tokens / Minute
```

TPM accounting for input/output tokens may differ based on provider rules.

### Complete Picture

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
Input       : What was given to the LLM?
Output      : What did the LLM generate?
Context     : How much information can the LLM handle in one request?
Latency     : How long did the response take?
Throughput  : How much work/tokens were processed per unit time?
Uptime      : How much time was the service available?
RPM         : How many requests allowed per minute?
TPM         : How many tokens allowed per minute?
TTFT        : How long did the first token take?
```

Rule of thumb: when selecting a production LLM, don't just look at intelligence/benchmark scores. Evaluate Quality, Context Window, Latency, Throughput, Rate Limits, Cost, and Uptime collectively.

---

## Hands-On Roadmap

These are practical build steps to follow after all the conceptual notes are clear. It might seem boring or difficult at first, but try to understand gradually and don't be scared by the number of code lines. There are only seven steps that need to be applied well.

### Step 1: Raw OpenAI SDK Fundamentals

- [x] Learn function calling
- [x] Understand tool schemas
- [x] Write a manual agent loop yourself: LLM call, tool_call detection, execute, result feed back, call again, final answer
- [x] Build a simple agent (weather tool) without any framework

Time estimate: 2-3 days

Status note: The full manual agent loop has been completed through the Client SDK with a weather tool. The complete cycle — LLM call, tool_call detection, execute, result feed back with `role: "tool"`, and final natural language answer from the second call — has been implemented in a spec-compliant way. Full code and explanation is in the OpenRouter Levels of Abstraction section.

Step 1 is fully complete. Next focus is the LangChain and LangGraph ecosystem (Step 3). CrewAI (Step 4) is deferred for now.

This is the most important step because every framework's concept is based on this.

### Step 2: RAG From Scratch

- [ ] Use a simple vector DB (Chroma or FAISS)
- [ ] Build your own document Q&A system
- [ ] Steps: chunking, embeddings, retrieval, then giving context to the LLM

Understanding how "memory" actually works is important here.

### Step 3: Learn LangGraph (for Control Flow)

- [ ] Build graph-based agents
- [ ] Understand nodes, edges, conditional routing, loops

This is the production-grade pattern used in real systems.

### Step 4: Try CrewAI for Multi-Agent

Status: deferred for now, will come back after LangGraph/LangChain ecosystem.

- [ ] Build a small project where 2-3 agents collaborate (researcher + writer)
- [ ] See how role-based abstraction works
- [ ] Notice where it feels rigid

### Step 5: Understand MCP and Build a Server

- [ ] Build a simple MCP server (like a custom tool wrapping an API)
- [ ] Connect it to Claude Desktop or any MCP client

This is the best way to understand how standardized tool integration works.

### Step 6: Explore Skills Pattern

- [ ] See how instructions are organized into modular files that the LLM loads dynamically, instead of one giant system prompt
- [ ] Try implementing this pattern in your own agent

### Step 7: Check n8n for Automation Angle

- [x] Try n8n (do this last)
- [x] By now you'll understand what's happening in the backend, so the visual tool will make more sense and you'll know where its limits are

---

*Last updated: 15 Aug 2026*
