---
tags: [ai, llm, providers, openrouter]
created: 2026-08-15
status: reference
---

# LLM Providers

## Providers kya hote hain

LLM providers wo companies hain jo apne trained models ko API ke through access karne dete hain. Har provider apni API key issue karta hai, aur uss key se hum unke models ko call kar sakte hain.

## Major LLM providers

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
- Amazon (Titan, Bedrock ke through multiple models)
- Microsoft Azure (Azure OpenAI Service)
- Together AI, Groq jaise infrastructure providers, jo doosron ke open models ko fast inference ke saath serve karte hain

Ye list poori nahi hai, lekin ye wo names hain jo abhi ke time par sabse zyada commonly use hote hain.

## Problem jo multiple providers use karne mein aati hai

Agar tumhe alag alag providers ke models try karne hain (jaise ek task OpenAI ke model se, dusra Anthropic se, teesra kisi open-source model se), to traditionally tumhe:

- Har provider ke saath alag account banana padta
- Har ek ki alag API key manage karni padti
- Har ek ka thoda different request/response format handle karna padta (jab tak wo OpenAI-compatible na ho)

Ye kaafi friction create karta hai jab tum experiment kar rahe ho ya multiple models compare kar rahe ho.

## Solution: aggregator platforms

Isi problem ko solve karne ke liye aggregator platforms bane, jo ek hi API key aur mostly consistent request format ke through, bahut saare providers ke models ko access karne dete hain. Inmein sabse jaana mana OpenRouter hai.

## Mera choice: OpenRouter

Main iss poore learning journey mein OpenRouter use karunga, kyunki:

- Ek hi API key se sainkdo models access ho jaate hain, alag alag providers ke
- Request/response format zyadatar OpenAI-compatible hai, jo already familiar hai
- Free tier models bhi available hain experiment karne ke liye
- Agar ek provider down ho ya rate limit hit ho, to fallback options bhi available hote hain

OpenRouter specifically kis tarah access diya jaata hai, teen alag levels of abstraction ke through, wo agle note mein detail mein cover kiya hai: [[04 OpenRouter Levels of Abstraction]].
