---
tags: [ai, openrouter, sdk, api]
created: 2026-08-15
status: reference
---

# OpenRouter: Levels of Abstraction

OpenRouter access dene ke teen levels set karta hai. Har level control aur convenience ke beech ek trade-off hai. Jitna zyada control chahiye, utna zyada code khud likhna padta hai, aur jitni zyada convenience chahiye, utna zyada implementation detail hidden ho jaata hai.

```text
Level 1: No SDK          -> Sabse zyada control, sabse zyada code
Level 2: Client SDK      -> Medium abstraction
Level 3: Agents SDK      -> Sabse zyada convenience, sabse kam control
```

## Level 1: No SDK (raw requests)

Iss level par koi bhi SDK use nahi kiya jaata, seedha `requests` library se OpenRouter ke REST endpoint ko HTTP call kiya jaata hai. Agar SDK use hi nahi karoge, to SDK se related koi problem bhi nahi hogi, bas code lamba likhna padega.

Fayda ye hai ki har ek part ke baare mein clearly pata rehta hai ki code kaise kaam kar raha hai, kyunki manual looping khud likhni padti hai, aur request/response ka har field khud dekhna padta hai.

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

Yahan `models` ek array ke roop mein pass kiya gaya hai, single `model` string ke bajaye. Isse OpenRouter ko ek fallback list mil jaati hai, agar pehla model available na ho ya fail ho jaaye, to next wala try kiya jaata hai.

### Response ka structure samajhna

Raw response ka JSON kuch is tarah aata hai:

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

Kuch important fields:

- `choices[0].message.content`, ye actual final answer hai jo user ko dikhana hai
- `choices[0].message.reasoning`, agar model reasoning-capable hai (jaise ye Nemotron model), to uska internal thinking process yahan aata hai, ye alag hota hai final content se
- `usage`, isme input/output tokens ka breakdown milta hai, aur agar reasoning use hua hai to uske tokens `completion_tokens_details.reasoning_tokens` mein alag se count hote hain
- `provider`, ye batata hai ki actually request kis underlying provider (jaise Nvidia) tak route hui thi

Ye sab fields raw response mein directly milte hain kyunki humne khud JSON parse kiya hai. Client SDK use karne par ye same data object-style access ke through milega, but underlying structure same hi rahega.

### Important security note

API key ko kabhi bhi directly code mein hardcode nahi karna chahiye, aur na hi kisi ke saath share karna chahiye. Hamesha `.env` file ya environment variable mein rakhna chahiye:

```python
import os
api_key = os.getenv("OPENROUTER_API_KEY")
```

## Level 2: Client SDK

Client SDK (jaise `openrouter` Python package, ya OpenAI SDK ko `base_url` change karke drop-in replacement ki tarah use karna) ek layer add karta hai upar se, taaki baar baar poora JSON manually na banana pade.

Deep down, SDK bhi wahi HTTP request kar raha hota hai jo Level 1 mein manually likhi thi. Bas ab wo complexity ek function call ke andar chhup gayi hai.

Pehla version jo banaya tha usme sirf ek round hota tha: LLM tool_call return karta, wo execute hota, aur bas wahin ruk jaata. Us se final natural language answer nahi milta tha. Poora, spec-compliant version isko ek loop mein daalta hai, jisme tool ka result wapas `messages` list mein `role: "tool"` ke saath feed hota hai, aur LLM ko dobara call kiya jaata hai jab tak wo final content na de de.

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

Yahan tools ka actual execution ek alag file (`tools_for_sdks.py`) mein define kiya gaya hai, jisme har tool asli API ko hit karta hai, jaise `get_weather` OpenWeatherMap ko call karta hai. LLM ne bas ye decide kiya ki kaunsa tool chahiye aur kya arguments hain, actual kaam (API hit karna) hamesha apne code ne kiya, exactly wahi principle jo [[02 Agents and Workflows]] mein cover hua tha.

### Loop ke andar kya ho raha hai, step by step

- `messages` ek list ki tarah loop se bahar define hoti hai aur har iteration mein usi list ko update kiya jaata hai, taaki poori conversation history preserve rahe
- Jab LLM `tool_calls` return karta hai, sabse pehle **LLM ka apna message** (`ai_message`) bhi `messages` mein add kiya jaata hai. Ye zaroori hai, warna spec ke hisaab se `role: "tool"` wala message kisi bhi assistant tool_call ke bina aata hai, jo invalid conversation state maana jaata hai
- Har tool call ka result `role: "tool"` ke saath add hota hai, saath mein `tool_call_id` bhi, taaki agar ek saath multiple tools call hue ho to LLM ko pata rahe kaunsa result kis call ka tha
- Loop tab tak chalta rehta hai jab tak LLM `tool_calls` na maange. Jaise hi LLM direct `content` de deta hai (matlab usko ab koi aur tool nahi chahiye), `else` branch trigger hoti hai, final answer print hota hai, aur `break` se loop terminate ho jaata hai

Ye poora structure hi wo "agentic behavior" hai jo [[02 Agents and Workflows]] mein describe kiya gaya tha. Frameworks (jaise CrewAI) internally yehi loop automate karte hain.

### Client SDK sirf chat tak limited nahi hai

Client SDK ka use sirf chat completions ke liye nahi hota. Jaise OpenAI SDK ko `base_url` change karke OpenRouter ke saath use kiya ja sakta hai audio transcription ke liye bhi:

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

Isse pata chalta hai ki Client SDK ek general purpose layer hai, jo chat ke alawa doosre AI services (transcription, embeddings, image generation, etc.) ke liye bhi wahi consistent pattern follow karta hai.

### Client SDK ka core idea

Jitna kam code likhenge, utne kam chances hain ki wo galat ho. Isliye Client SDK traditional tareeke se hi kaam karta hai, RESTful APIs ke through, lekin code ko chhota kar deta hai jisse implementation aasaan ho jaata hai. Underlying request wahi rehta hai jo Level 1 mein manually likha tha.

## Level 3: Agents SDK

Agents SDK (jaise OpenRouter ka `@openrouter/agent` package) sabse zyada abstraction deta hai. Ye poora multi-turn loop, jo Level 1 aur Level 2 mein manually likhna padta tha (LLM call, tool_call detect, execute, result feed back, dobara call), internally khud handle kar leta hai, ek single function call ke through (jaise `callModel`).

Iska ek limitation abhi ke liye ye hai ki ye sirf TypeScript/JavaScript mein supported hai, Python mein nahi. Isliye maine ye try nahi kiya hai, kyunki abhi tak Python-based Client SDK aur No SDK approach hi use kiya hai.

## Teeno levels ka comparison

| Level | Control | Code Amount | Language | Status |
|---|---|---|---|---|
| No SDK | Sabse zyada | Sabse zyada | Koi bhi (raw HTTP) | Try kiya hai |
| Client SDK | Medium | Medium | Python, JS, aur jahan bhi SDK ho | Try kiya hai |
| Agents SDK | Sabse kam | Sabse kam | Sirf TypeScript/JavaScript | Try nahi kiya |

Agla topic ye hai ki tools, MCP, aur skills actually LLM ke saath kaise interact karte hain, jo teeno levels mein common concept hai: [[05 Tools MCP and Skills]].
