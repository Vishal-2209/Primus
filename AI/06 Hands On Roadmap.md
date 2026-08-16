---
tags: [ai, agents, roadmap, learning]
created: 2026-08-15
status: in-progress
---

# Hands On Roadmap

Ye practical build steps hain, jo saare conceptual notes ([[01 Machine Learning and LLM Fundamentals]] se [[05 Tools MCP and Skills]] tak) clear hone ke baad follow karne hain. Shuruaat mein sab boring ya mushkil lagta hai, lekin dheere dheere samajhne ki koshish karo aur code ki lines dekh kar mat daro. Sirf saat steps hain jinko achhe se apply karna zaroori hai.

## 1. Raw OpenAI SDK se fundamentals

- [x] Function calling seekho
- [x] Tool schemas samjho
- [x] Manual agent loop khud likho, poora cycle: LLM call, tool_call detect, execute, result feed back, dobara call, final answer
- [x] Simple agent banao (weather tool) bina kisi framework ke

Time estimate: 2-3 din

Status note: Client SDK ke through complete manual agent loop ban chuka hai, weather tool ke saath. LLM call, tool_call detect, execute, result `role: "tool"` ke saath wapas feed karna, aur final natural language answer aana tak dobara call karna, ye poora cycle spec-compliant tareeke se implement ho chuka hai. Poora code aur explanation [[04 OpenRouter Levels of Abstraction]] mein hai.

Step 1 poora complete. Ab agla focus LangChain aur LangGraph ecosystem hai (Step 3), CrewAI (Step 4) ko filhal baad ke liye rakha hai.

Ye sabse zaroori step hai kyunki har framework ka concept isi par based hai.

## 2. RAG implement karo from scratch

- [ ] Simple vector DB use karo (Chroma ya FAISS)
- [ ] Apna document Q&A system banao
- [ ] Steps: chunking, embeddings, retrieval, aur phir LLM ko context dena

Ye samajhna zaroori hai ki "memory" actually kaise kaam karta hai.

## 3. LangGraph seekho (control flow ke liye)

- [ ] Graph-based agents banao
- [ ] Nodes, edges, conditional routing, loops samjho

Yahi production-grade pattern hai jo real systems mein use hota hai.

## 4. CrewAI try karo multi-agent ke liye

Status: filhal deferred, LangGraph/LangChain ecosystem ke baad wapas aayenge.

- [ ] Chhota project banao jisme 2-3 agents collaborate karte hon (researcher plus writer)
- [ ] Dekho ki role-based abstraction kaise kaam karta hai
- [ ] Observe karo ki kaha ye rigid feel hota hai

## 5. MCP samjho aur ek server banao

- [ ] Simple MCP server banao (jaise ek custom tool jo kisi API ko wrap kare)
- [ ] Use Claude Desktop ya kisi MCP client mein connect karo

Ye samajhne ke liye best hai ki standardized tool integration kaise kaam karta hai.

## 6. Skills pattern explore karo

- [ ] Dekho ki instructions ko modular files mein kaise organize karte hain jo LLM dynamically load kare, instead of ek giant system prompt
- [ ] Apne khud ke agent mein ye pattern implement karke try karo

## 7. n8n se automation angle dekho

- [ ] n8n try karo (sabse last mein)
- [ ] Ab tumhe pata hoga backend mein kya chal raha hai, toh visual tool bhi zyada sense banayega aur tum jaanoge kaha iski limits hain

## Related Notes

- [[00 Master Plan]]
- [[04 OpenRouter Levels of Abstraction]]
- [[05 Tools MCP and Skills]]

## Tags

#ai #agents #langgraph #crewai #mcp #n8n #roadmap
