---
Projects Covered:
  - Lawprix
  - Nexus
  - PGPulse
created: 2026-08-30
---
Built from your resume, the case studies on va7.dev, and your project history. Organized by project, then cross-project system design, ML/AI fundamentals (given the GenAI/ML component of your round), and a section that pressure-tests the quantified claims on your resume.
%%## Before you go in: one thing to reconcile

Your resume describes Nexus as a Django backend with Firebase Auth and Supabase file storage. The live case study on va7.dev tells a different, more detailed story: a Flask backend acting as a zero-auth caching proxy in front of the Google Drive API, no login at all, with a TTL-based in-memory cache and specific metrics (98% cache hit ratio, sub-50ms cached responses). These can't both be the current architecture. The case study version is far more technically specific, so it reads like the real build, but you should confirm which one is actually true before an interviewer asks you to whiteboard it. The Nexus questions below are written around the case study's zero-auth, caching-proxy version since that's the one with real engineering depth to discuss, but hold it loosely until you've checked.%%

---

## LawPrix — Legal Tech SaaS Platform

### Architecture & System Design

1. Walk me through the overall architecture, portal by portal. How do the client, business, and lawyer portals share code versus stay isolated?
2. Why 22 separate Django apps instead of fewer, larger ones? What's your rule of thumb for when something earns its own app?
3. All three portals sit behind a single DRF API layer. How do you guarantee a client-side user can never hit a lawyer-only or admin-only endpoint?
4. If you had to add a fourth portal type tomorrow, say court staff, what would you need to touch across the 22 apps?
5. Walk me through what happens end to end when a client submits a new case, from the API call to it landing in front of a matched lawyer.

### Backend, API & Access Control

6. Explain how your JWT-based role-level access control actually works: access token vs refresh token, where they live client-side, what happens on expiry.
7. If a lawyer's verification status changes mid-session, does their JWT get revoked immediately, or do they keep old permissions until it expires?
8. How do you version your DRF APIs as the platform evolves? Have you had to make a breaking change yet?

### Machine Learning — Case Routing

9. The case-routing engine is a scikit-learn classification pipeline on case type, urgency, and lawyer specialization. Walk me through the pipeline: what does training data look like, and where does it actually come from on a live platform?
10. Why a classifier here rather than a rules-based scoring function, given the inputs sound like they could be hand-weighted?
11. How do you handle a new case type or a lawyer specialization the model has never seen, a cold start problem?
12. Since the whole pitch is eliminating pay-to-rank bias, how do you monitor the model itself for developing its own bias, like always routing to the same handful of lawyers?
13. What happens when the model makes a confident but wrong match? Is there any human review or override path?

### LLM & RAG — Case Assessment Engine

14. Walk me through the RAG pipeline for case assessment: what gets embedded, where it's stored, and what the retrieval step actually pulls back.
15. Why NVIDIA Nemotron and OpenAI both, routed through OpenRouter, instead of committing to a single provider?
16. A hallucinated case summary is a real liability in a legal product. What guardrails sit between the LLM's output and a lawyer actually reading it?
17. How do you evaluate the quality of the case assessment output? Any human-in-the-loop review, or ground truth to check against?
18. What's your prompting approach, structured output or free text, and what happens when a case doesn't fit the expected shape?

### Async Processing (Celery + Redis)

19. Why does case assessment need Celery and Redis rather than running inline on the request?
20. Walk me through the task lifecycle: a case comes in, a task gets queued, and it fails halfway through. What's your retry behavior, and how does the client find out the analysis is ready?

### Data Layer

21. Why Supabase as the primary data layer instead of a self-managed Postgres instance? What does it actually buy you?
22. How is multi-tenancy modeled at the database level: one schema with a tenant column, or real isolation between client, business, and lawyer data?

### Security

23. Tell me about the security audit with the Antigravity coding agent. What did the Tier 1 vs Tier 2 authority framework actually let it do autonomously versus require your sign-off?
24. What's the most serious vulnerability that audit surfaced, and what was the root cause, not just the fix?
25. It also caught a long-standing bug in the matching engine. How long had it been live, and what was the real-world impact, were cases actually being mis-routed?
26. Having now used an AI agent for a live production audit, where's your line for what you'd trust it to change autonomously versus always requiring a human?

### Product Evolution & Judgment

27. LawPrix started as Law Connect / Vakaalat, a Flask app with a scikit-learn classifier. What specifically pushed a full rebuild on Django rather than extending the Flask version?
28. What did the SSIP Gujarat grant actually require in terms of milestones, and where does the platform stand against that today?
29. Where's LawPrix currently deployed, and why that choice over the alternatives?

---

## PGPulse — Multi-Tenant PG Management System

### Architecture & Infrastructure

30. You built this solo in five weeks and chose a containerized microservices architecture. Isn't that a lot of infrastructure overhead for a five-week solo build? Why not a monolith given the timeline?
31. What are the actual service boundaries? Is the AI inference pipeline, face recognition and meal prediction, a separate service from the core Django app, and how do they talk to each other?
32. What's running in each container, and how are they orchestrated?

### Face Recognition Pipeline

33. Walk me through the pipeline end to end: the webcam captures a frame, and then what extracts the 512-dimensional embedding, client or server?
34. Once you have an embedding, how do you match it against stored residents? What similarity metric, and what threshold decides a match versus a stranger?
35. How do you defend against someone holding up a photo instead of their actual face? Any liveness detection?
36. Where and how are the facial embeddings stored? What would a breach of that table actually expose, given embeddings can sometimes be reversed toward an approximate face?
37. What's your false accept vs false reject rate in practice, and which one did you optimize for?

### AI Meal Prediction

38. The meal predictor is a Random Forest classifier predicting counts. Counts sound like a regression target. Why classification instead?
39. Walk me through the features: historical attendance, weather, day of week. How far back does "historical" go, and how do you handle a brand-new branch with no history?
40. How do you evaluate this model? What's the error metric, and what does a bad prediction actually cost, wasted food or a shortage?
41. Weather is a feature. What's the actual signal there, and how confident are you that's real and not noise, given the five-week build window?
42. How often does it retrain, and what happens to prediction quality between retrains as a branch's population shifts?

### Multi-Tenant RBAC

43. Owners switch between PG branches without separate logins. Walk me through the implementation: single session with a tenant-context switch? How do you guarantee a query never leaks across branches?
44. What roles exist beyond owner, and how granular is the permission model?

### Frontend State & Inventory

45. You're using both React Query and Zustand. What's the dividing line between them, and has anything ever gotten out of sync?
46. "Real-time state" is mentioned. What's actually pushing updates: polling, or something like websockets?
47. Low-stock alerts are "dynamically configured." Configured by whom, and based on what? Is there any predictive element, or is it a static per-item minimum?

---

## Nexus — Zero-Auth Resource Aggregation Platform

_(See the flag at the top before this section. Questions below assume the case-study architecture.)_

### Architecture & Trust Model

48. Walk me through the zero-auth design. The backend proxies Google Drive metadata without exposing your API key to the client. What would actually break if you skipped the proxy and called Drive directly from React?
49. With no login at all, how do you prevent abuse, like someone scraping the endpoint or burning through your Drive API quota?
50. If someone reverse-engineered your REST endpoint, what's the worst they could actually do?

### Caching Strategy

51. You report roughly a 98% cache hit ratio with a TTL-based in-memory cache. Walk me through the mechanics: what's the TTL, and why that number?
52. What happens on a cache miss during peak exam-season traffic, if a large number of students hit an expired key at nearly the same moment? Could that cause a thundering herd against the Drive API?
53. In-memory cache implies it lives in the process. If you ever scaled to multiple backend instances behind a load balancer, what breaks, and how would you fix it?
54. You serve stale content if the Drive API is unreachable. How stale is too stale before you'd rather show an error than wrong data?
55. Your numbers show 300 to 500ms cold versus sub-50ms cached. How did you actually measure that?

### Google Drive Integration & Scale

56. How do folders map to your semester and course-category filters, structurally by folder naming, or via metadata?
57. What Drive API rate limits did you actually hit while building this, and how close to the free quota are you at 700 monthly active users?
58. The 1,200 concurrent users number is specific. How was that measured?
59. Zero marketing spend to 700 organic MAU is a strong result. What do you think actually drove that, and would it have worked without the zero-auth design specifically?

---

## Cross-Project System Design Questions

60. You've built a modular Django monolith (LawPrix, 22 apps), a containerized microservices system (PGPulse), and a thin caching proxy (Nexus). What's your actual decision process for picking an architecture style? Would you build any of the three differently today?
61. Auth strategy differs across all three. What's your framework for deciding whether an app needs auth at all, and what kind?
62. LawPrix uses Supabase, PGPulse uses plain PostgreSQL. When do you reach for a managed backend-as-a-service versus rolling your own?
63. When do you reach for Celery and Redis versus a simpler synchronous or in-memory approach? Where's the actual threshold in your head?
64. None of the three write-ups mention automated testing. What's your testing strategy across them, and if there isn't one yet, how would you retrofit tests onto LawPrix specifically, given it's live with real user data?
65. How do you manage secrets and environment config across three separately deployed projects?
66. You list Django, Flask, and FastAPI as frameworks you've used. When would you reach for FastAPI over both of the others, and is there anything in these three projects you'd rebuild in it today?

## AI/ML Fundamentals They're Likely to Probe

67. What is RAG, in your own words, and where does it actually break down compared to fine-tuning a model on your own case data?
68. Explain embeddings to someone non-technical, then explain what "512-dimensional" is actually a knob on.
69. Cosine similarity versus Euclidean distance for comparing face embeddings: which do you use, and why does it matter?
70. Explain a Random Forest from first principles: why does an ensemble of trees beat a single decision tree, and what does "the trees vote" mean at inference time?
71. Precision versus recall. Across your three ML components, case routing, meal prediction, face recognition, which one matters more for each, and why?
72. What's overfitting, and what would tell you your meal-prediction model is overfit to one branch's history?
73. How would you actually A/B test a change to the case-routing model on a live platform, where a bad match has a real-world cost?

## Pressure-Testing Your Resume Claims

74. "Client onboarding time dropped" at Atodya. Dropped by how much, and what's the source of that number, timed measurement or general impression?
75. "UI development time... reduced" at Meru Technosoft. Same question: what's the actual before-and-after?
76. "Developer handoff friction dropped." How do you measure friction, and what concretely changed in the handoff process?
77. Landing pages at MCS Cargar, "improved performance" via lazy loading and code splitting. What were the real before-and-after numbers, Lighthouse score, load time?
78. Your summary claims "2+ years of internship and freelance experience." Walk me through how those years stack up given some of the internships overlap.