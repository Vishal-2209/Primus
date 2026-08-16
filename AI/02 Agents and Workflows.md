---
tags: [ai, agents, workflows, frameworks]
created: 2026-08-15
status: reference
---

# Agents and Workflows

## Asli topic ye hai

LLMs ka mechanism (word prediction, tokenization, prompt execution) samajhne ke baad asli sawaal ye hai: in LLMs ka use karke hum cheezon ko practically better kaise bana sakte hain. Iska jawab hai, agents aur workflows design karke.

## Design karna aur banana, do alag tasks hain

Ek agent ya workflow ko design karna (yani ye plan karna ki kya steps honge, kaunse tools chahiye honge, kahan decision points honge) ek cheez hai. Usko actually implement karke banana ek dusra, alag task hai.

Ye implementation traditional coding aur DSA (data structures and algorithms) use karke bhi kiya ja sakta hai, poori tarah manually. Lekin practically log alag alag "agent frameworks" use karte hain, taaki:

- Repetitive boilerplate code baar baar na likhna pade
- Common patterns (loop, tool execution, state management) already ready milein
- Implementation thoda aasaan aur standardized ho jaaye

Frameworks (jaise LangGraph, CrewAI) is roadmap ke aage wale steps mein cover honge: [[06 Hands On Roadmap]].

## Ek agent actually hota kya hai

Ye sabse important mental model hai jo clear hona chahiye: **ek agent koi alag, magical entity nahi hota. Ek agent basically ek LLM hi hota hai**, jise kuch tools available karaye jaate hain, aur jab zaroorat padti hai, wo un tools ko use karne ke liye ek required JSON schema generate karta hai (yani tool ka naam aur uske parameters, structured format mein).

```text
Agent = LLM + Tool Definitions (JSON schemas)
```

Jab hum bolte hain "agent ne decide kiya ki weather check karni hai", to actually LLM ne bas itna kiya hai ki usne ek JSON object generate kiya hai jisme likha hai:

```json
{
  "tool": "get_weather",
  "arguments": { "city": "Ahmedabad" }
}
```

LLM khud us tool ko execute nahi karta. LLM sirf itna decide karta hai ki kaunsa tool chahiye aur kya arguments dene hain. Actual execution hamesha bahar, hamare apne code mein hota hai.

## Toh "agentic" behavior aata kahan se hai

Agar LLM sirf ek schema generate karta hai, to poora multi-step, tool-using behavior kaise banta hai. Iska jawab hai: ek loop.

```text
LLM ko call karo (tools ke saath)
        ↓
Check karo: LLM ne tool_call return kiya ya direct answer diya
        ↓
Agar tool_call hai:
    Tool ko actually execute karo (apna code)
    Result ko wapas conversation mein feed karo
    LLM ko dobara call karo
        ↓
Ye loop tab tak chalta hai jab tak final answer na mil jaaye
```

Ye wahi manual loop hai jo raw SDK ke saath likha jaata hai, aur yahi loop hai jo agent frameworks (LangGraph, CrewAI, ya OpenRouter ka Agents SDK) apne andar automate kar dete hain. Framework use karne ka fayda ye hai ki ye loop, state management, aur error handling khud handle ho jaata hai, lekin underneath concept bilkul same rehta hai jo manually likha gaya tha.

## Summary

- LLM sirf prediction aur schema generation karta hai
- Execution hamesha external code (hamara apna, ya framework ka internal code) karta hai
- "Agent" ek label hai us pure setup ke liye: LLM plus tools plus loop
- Frameworks is setup ko implement karna aasaan banate hain, lekin naya concept nahi hai

Agla step ye samajhna hai ki in LLMs tak access kaise milta hai, kaunse providers available hain, aur specifically OpenRouter kis tarah kaam karta hai: [[03 LLM Providers]].
