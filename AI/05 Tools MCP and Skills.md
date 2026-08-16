---
tags: [ai, tools, mcp, skills]
created: 2026-08-15
status: reference
---

# Tools, MCP, and Skills

Ye teeno concepts alag alag lagte hain, lekin inke peeche ek hi common principle hai: **LLM sirf ek schema ya decision generate karta hai, actual execution hamesha bahar, code ke through hota hai.** Ye samajhna sabse zaroori cheez hai, kyunki ye tino cheezein isi principle ke upar based hain.

## Tools

Ek tool basically ek function ka description hota hai, jo LLM ko diya jaata hai JSON schema ke format mein. Isme hota hai:

- Tool ka naam
- Description (LLM ko batata hai ki ye tool kya karta hai)
- Parameters (kaunse arguments chahiye, kis type ke)

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

Jab LLM ko lagta hai ki user ke sawaal ka jawab dene ke liye is tool ki zaroorat hai, to LLM us tool ko khud call nahi karta. LLM sirf itna output deta hai:

```json
{
  "name": "get_weather",
  "arguments": { "city": "Ahmedabad" }
}
```

Iske baad hamara apna code is JSON ko parse karta hai, actual function ko call karta hai (jo real API ko hit karega, jaise OpenWeatherMap), aur result ko wapas LLM tak feed karta hai agar final natural language answer chahiye.

## MCP (Model Context Protocol)

Agar har project mein tools ke schemas aur unka execution logic alag alag likhna pade, to ye repetitive ho jaata hai, aur har framework/client ka apna tool-integration format hone se cheezein aur bhi inconsistent ho jaati hain.

MCP ek standard protocol hai jo isi problem ko solve karta hai. Isse tool definitions aur execution logic ko ek "server" ke roop mein ek baar package kiya ja sakta hai, aur phir wo server kisi bhi MCP-compatible client (Claude Desktop, ya koi bhi doosra MCP client) mein plug ho sakta hai, bina dobara se schema aur logic likhne ki zaroorat ke.

Analogy: jaise USB-C port sab devices ke liye ek common standard ban gaya hai, waise hi MCP tool-integration ke liye common standard hai.

```text
Bina MCP: Har client apna alag tool format define karta hai
Saath MCP: Ek MCP server banao, kisi bhi MCP client mein plug karo
```

Yahan bhi principle wahi hai: MCP server ke andar bhi LLM sirf decide karta hai ki kaunsa tool chahiye, actual execution MCP server ke code mein hota hai, LLM ke andar nahi.

## Skills

Skills ek relatively naya pattern hai (jaise Claude Skills), jisme instructions ko ek hi bade system prompt mein daalne ke bajaye, chhote chhote modular instruction files mein organize kiya jaata hai. LLM ko in files ka sirf ek short description pata hota hai upfront, aur jab koi specific task relevant lagta hai, tab wo poori file dynamically load karta hai.

Ye pattern "progressive disclosure" kehlaata hai, matlab sab kuch upfront context mein daalne ke bajaye, sirf tab load karo jab zaroorat ho.

```text
Bina Skills: Ek hi giant system prompt, sab kuch hamesha context mein
Saath Skills: Chhoti modular files, sirf relevant wali load hoti hai jab zaroorat ho
```

Yahan bhi wahi principle apply hota hai. LLM khud file ko load karne ka mechanism nahi chalata, wo sirf decide karta hai ki "mujhe is naam ki skill relevant lagti hai", aur baaki ka loading, file reading, aur actual use, surrounding code/system handle karta hai.

## Common thread

Tools, MCP, aur Skills, teeno mein ek hi cheez repeat ho rahi hai:

```text
LLM ka kaam: decide karna aur schema generate karna
Code ka kaam: us decision ko actually execute karna
```

Ye mental model poore agent-building process mein baar baar kaam aayega, chahe raw SDK se manual loop likhna ho, chahe LangGraph mein graph banana ho, ya chahe MCP server design karna ho. LLM kabhi bhi khud kuch "execute" nahi karta, wo sirf structured decisions deta hai.

Ab jo concepts clear ho chuke hain, unke upar hands-on practical roadmap follow karna hai: [[06 Hands On Roadmap]].
