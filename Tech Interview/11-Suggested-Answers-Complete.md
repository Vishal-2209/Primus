---
created: 2026-08-30
purpose: Complete suggested answers for PGPulse, Nexus, Cross-Project, AI/ML, and Resume questions
---
# Suggested Answers - Complete (Q30-Q78)

> Companion to the questions file. Answers grounded in your project details. Fill in **[brackets]** with your real numbers/implementation details.

---

## PGPulse - Multi-Tenant PG Management System

**Q30. You built this solo in five weeks and chose a containerized microservices architecture. Isn't that a lot of infrastructure overhead for a five-week solo build? Why not a monolith?**

A: Fair challenge. The monolith would have been faster to ship initially, but the AI inference pipeline (face recognition + meal prediction) is compute-intensive and has different resource requirements than the core Django app. Separating them meant I could scale the AI service independently without over-provisioning the web server. Docker made local development reproducible, and the boundaries were clean enough that I wasn't spending time on inter-service complexity. If I were rebuilding today for the same timeline, I'd start monolith and extract the AI service only when I hit scaling pressure.

**Q31. What are the actual service boundaries? Is the AI inference pipeline a separate service from the core Django app?**

A: Yes. The core Django service handles all CRUD operations - residents, inventory, subscriptions, billing. The AI service handles face recognition inference and meal prediction. They communicate via REST API calls - Django calls the AI service for inference, and the AI service pulls training data from Django's database. This separation meant I could deploy model updates to the AI service without touching the stable Django codebase.

**Q32. What's running in each container, and how are they orchestrated?**

A: Container 1: Django + Gunicorn serving the REST API. Container 2: AI inference service with InsightFace and scikit-learn models loaded in memory. Container 3: PostgreSQL database. Container 4: Redis for caching and session storage. I used Docker Compose for orchestration during development **[and Kubernetes/Docker Swarm for production if applicable]**.

---

### Face Recognition Pipeline

**Q33. Walk me through the pipeline end to end: the webcam captures a frame, and then what extracts the 512-dimensional embedding?**

A: The webcam captures a frame via the browser's MediaDevices API. The frame is sent as base64 to the Django backend. Backend uses InsightFace's ArcFace model to detect the face and extract a 512-dimensional embedding vector. This happens server-side to keep the model weights off the client. The embedding is then compared against stored embeddings in the database for matching.

**Q34. Once you have an embedding, how do you match it against stored residents? What similarity metric and threshold?**

A: I use cosine similarity to compare the new embedding against all stored embeddings for that PG branch. Cosine similarity measures the angle between vectors, which is more robust than Euclidean distance for high-dimensional embeddings. **[State your threshold, e.g., "A similarity score above 0.6 is considered a match"]**. If no stored embedding exceeds the threshold, the person is flagged as unrecognized.

**Q35. How do you defend against someone holding up a photo instead of their actual face? Any liveness detection?**

A: **[State honestly. If not implemented: "Currently there's no liveness detection. This is a known limitation. The system relies on the webcam context - it's used at the PG entrance where staff presence provides a physical security layer. For a production deployment, I'd add liveness detection using blink detection or depth analysis."]**

**Q36. Where and how are the facial embeddings stored? What would a breach expose?**

A: Embeddings are stored in PostgreSQL as binary vectors in a dedicated table linked to the resident profile. **[State your encryption approach. If not encrypted at rest: "Currently not encrypted at rest, which is a valid concern. Embeddings are 512 floating point numbers - they can't be reversed to reconstruct a recognizable face easily, but they could potentially be used for impersonation against systems that use the same embedding model. For production, I'd encrypt the embedding column and restrict access via row-level permissions."]**

**Q37. What's your false accept vs false reject rate? Which did you optimize for?**

A: **[Provide real numbers if you tested. If not formally measured: "I optimized for false rejects over false accepts - it's better to ask a resident to try again than to incorrectly grant attendance to a stranger. In manual testing with the residents at the PG, I observed roughly [X] false rejects out of [Y] attempts, and zero false accepts. Formal metrics would require a larger test set."]**

---

### AI Meal Prediction

**Q38. The meal predictor is a Random Forest classifier predicting counts. Counts sounds like a regression target. Why classification instead?**

A: Good catch. I framed it as classification because the output needs to be a discrete number of meals - you can't prepare 37.5 meals. Classification naturally outputs whole numbers. The features (day of week, weather, historical attendance patterns) create distinct categories that the model can learn. Random Forest handles the non-linear relationships between weather and attendance well. A regression approach would require rounding anyway, and the classification model performed better in practice on this dataset size.

**Q39. Walk me through the features. How far back does historical data go?**

A: Features include: day of week (one-hot encoded), weather conditions (temperature, rain/sunny), branch ID, historical average attendance for that day, and occupancy rate. Historical data goes back **[state your actual window]**. For a new branch with no history, I use the average across all branches as a fallback until enough data accumulates **[mention the threshold, e.g., "after 2 weeks of data, it switches to branch-specific predictions"]**.

**Q40. How do you evaluate this model? What's the error metric?**

A: I use MAE (Mean Absolute Error) as the primary metric - it directly tells me how many meals off the prediction is on average. A bad prediction means either food waste (over-prediction) or shortage and unhappy residents (under-prediction). **[State your actual MAE. If not measured: "In manual testing, predictions were typically within 2-3 meals of actual consumption, which I considered acceptable for a 5-week build."]**

**Q41. Weather is a feature. What's the actual signal there?**

A: The hypothesis is that hot days reduce appetite and rainy days keep people indoors (more likely to eat at the PG). **[State your confidence honestly: "Given the five-week build window, I had limited data to validate this. The feature importance from Random Forest showed weather ranked **[state rank]**, so it contributes but isn't the dominant signal. Day of week and historical average were stronger predictors."]**

**Q42. How often does it retrain? What happens between retrains?**

A: **[State your actual retraining schedule. If manual: "Currently retraining is manual - I trigger it when I notice prediction quality degrading. For a production system, I'd set up weekly retraining on a cron job, using the most recent data."]**

---

### Multi-Tenant RBAC

**Q43. Owners switch between PG branches without separate logins. Walk me through the implementation.**

A: The JWT token contains the user's ID and role (owner). When an owner selects a different branch, the frontend sends a context-switch request that updates a `current_branch` field in the session. Every subsequent query includes this branch context, and the backend queryset filtering ensures data never leaks across branches. The permission model checks: (1) Is this user an owner? (2) Does this branch belong to this owner? (3) Is the requested data within this branch's scope?

**Q44. What roles exist beyond owner?**

A: **[List your actual roles. Example: Owner (full access across branches), Manager (branch-level management), Staff (limited operations like attendance marking), Resident (view own data). Permission granularity: owners can manage billing and subscriptions, managers handle day-to-day operations, staff only mark attendance.]**

---

### Frontend State & Inventory

**Q45. You're using both React Query and Zustand. What's the dividing line?**

A: React Query handles server state - API calls, caching, synchronization, background refetching. Zustand handles client state - UI state like sidebar open/closed, selected branch context, form state. The dividing line is: does this data come from the server? Use React Query. Is it purely client-side UI state? Use Zustand.

**Q46. What's actually pushing updates: polling or websockets?**

A: **[State your actual implementation. If polling: "Currently polling with React Query's refetchInterval for real-time updates. For a production version, I'd move to WebSockets for true real-time, especially for the attendance dashboard."]**

**Q47. Low-stock alerts are "dynamically configured." Configured by whom?**

A: **[State your actual implementation. Example: "Owners set minimum thresholds per inventory item through the dashboard. Alerts trigger when stock drops below the threshold. There's no predictive element yet - it's purely threshold-based. A predictive approach would analyze consumption patterns and predict when stock will run out."]**

---

## Nexus - Zero-Auth Resource Aggregation Platform

**Q48. Walk me through the zero-auth design. What breaks if you skip the proxy?**

A: If I called Drive directly from React, the Google API key would be exposed in client-side JavaScript. Anyone could extract it and make unlimited API calls, burning through quota or accessing any public Drive folder I've connected. The backend proxy keeps the API key server-side, acts as a trust boundary, and allows me to enforce rate limiting and caching at the application level.

**Q49. How do you prevent abuse without login?**

A: Several layers: (1) Rate limiting per IP address to prevent scraping. (2) Caching means even high traffic doesn't translate to proportional Drive API calls. (3) The backend only exposes file metadata (name, size, thumbnail URL) - not the actual file contents or Drive folder structure. (4) **[Mention any other measures you implemented]**. The trade-off is accepting some abuse risk for zero-friction access.

**Q50. If someone reverse-engineered the REST endpoint, what's the worst they could do?**

A: They could enumerate all files across all semesters and courses by querying different filter combinations. They couldn't access actual file contents (that's controlled by Google Drive's public permissions). They could potentially cause cache misses by requesting non-existent combinations. The rate limiter caps the damage.

**Q51. What's the TTL, and why that number?**

A: **[State your actual TTL. Example: "TTL is set to **[X minutes/hours]**. I chose this based on how often course materials change - during semesters, files are added frequently so TTL is shorter. During breaks, longer TTL is fine. The 98% hit ratio suggests the TTL aligns well with access patterns."]**

**Q52. What happens on cache miss during peak traffic? Could it cause thundering herd?**

A: This is a real concern. If 500 students hit an expired key simultaneously, all 500 would trigger Drive API calls. My implementation uses a simple lock mechanism - **[describe if you have one. If not: "Currently there's no thundering herd protection. For production, I'd implement request coalescing - the first request fetches from Drive while others wait, then all get the cached result."]**

**Q53. If you scaled to multiple instances, what breaks?**

A: In-memory cache is per-process. If I have 3 instances behind a load balancer, each has its own cache, reducing hit ratio. Fix: use a shared cache layer like Redis, or a CDN with cache invalidation. The Flask app would need to be stateless, which it already is since the cache is the only state.

**Q54. How stale is too stale?**

A: For academic resources, stale data means showing files that were deleted or missing new uploads. A student clicking a link to a deleted file is a bad experience but not dangerous. I'd show stale content up to **[state your threshold]** and then show an error with a "resources may have changed" message. For a legal product like LawPrix, stale would be unacceptable - different context, different threshold.

**Q55. How did you measure 300-500ms cold vs sub-50ms cached?**

A: **[State your measurement approach. Example: "I logged response times for each request over a 2-week period, categorizing by whether the request hit cache or triggered a Drive API call. Cold requests averaged 350ms, cached requests averaged 45ms. The 98% cache hit ratio came from counting cache hits vs total requests."]**

**Q56. How do folders map to filters?**

A: Folder naming convention: **[describe your structure. Example: "Folders are named `Semester_CourseCategory_ResourceType` - e.g., `S3_CS_Notes`, `S5_ECE_Labs`. The backend parses folder names to build the filter taxonomy. Metadata isn't used since Drive folder metadata is limited."]**

**Q57. What Drive API rate limits did you hit?**

A: Google Drive API allows **[state quota]** requests per day. At 700 MAU with 98% cache hit ratio, actual API calls are minimal - roughly **[calculate: e.g., 700 users * 20 requests/day * 2% cache miss = 280 API calls/day]**, well within free tier limits.

**Q58. How was 1,200 concurrent users measured?**

A: **[State honestly. Example: "During the exam season at PDEU, I monitored server metrics and saw **[X]** concurrent connections. The number was measured via **[server logs / analytics / monitoring tool]**."]**

**Q59. What drove the organic growth?**

A: The zero-auth design is the primary driver. Students share links directly - no signup friction, no "download our app" barrier. A WhatsApp group message with a link works immediately. The speed (sub-50ms responses) makes it feel native. Word-of-mouth among PDEU students during exam season amplified it. Without zero-auth, conversion would drop significantly - every login wall loses users.

---

## Cross-Project System Design Questions

**Q60. What's your decision process for architecture style?**

A: I think about three things: (1) Complexity of business logic - LawPrix has 22 apps with complex RBAC, so Django monolith with app separation makes sense. (2) Resource heterogeneity - PGPulse's AI pipeline needs different scaling than CRUD, so microservices. (3) User trust model - Nexus needs zero auth, so thin proxy is sufficient. If rebuilding today, I might make LawPrix a modular monolith with the option to extract services, and I'd definitely add automated testing to all three.

**Q61. What's your framework for deciding auth strategy?**

A: Questions I ask: (1) Is the data sensitive? LawPrix handles legal cases - needs full auth. (2) Is there a business reason to know who the user is? PGPulse needs to track attendance. (3) Does auth add value or friction? Nexus - auth adds friction for students who need quick access. (4) What's the cost of abuse? Nexus accepts some abuse risk for zero-friction access.

**Q62. When do you reach for managed BaaS vs rolling your own?**

A: Managed (Supabase, Firebase) when: building solo or small team, need to ship fast, core differentiator isn't the database layer. Self-managed when: need full control, have ops expertise, data requirements are unique, or cost becomes prohibitive at scale. LawPrix uses Supabase because the differentiator is the ML routing, not the database. PGPulse uses PostgreSQL because the AI pipeline needed direct database access for feature engineering.

**Q63. When do you reach for Celery vs synchronous?**

A: Celery when: (1) Operation takes >500ms and user shouldn't wait, (2) Operation might fail and needs retry logic, (3) Operation is expensive (LLM calls, model inference). Synchronous when: operation is fast, result is needed immediately, and failure should be immediately visible to the user.

**Q64. What's your testing strategy?**

A: **[Honest answer. Example: "Testing is a gap across all three projects. For LawPrix, I'd prioritize: (1) Integration tests for the case routing pipeline, (2) Permission tests to ensure role-based access control works, (3) API contract tests for the DRF endpoints. I'd use pytest with DRF's APITestCase for backend, and Jest for React frontend."]**

**Q65. How do you manage secrets?**

A: **[State your approach. Example: "Environment variables via .env files (not committed to git), with different configs for development and production. Supabase and Firebase credentials are in environment variables. API keys for OpenRouter are server-side only. For production, I'd use a secrets manager like AWS Secrets Manager or Vault."]**

**Q66. When would you reach for FastAPI?**

A: FastAPI when: (1) Building a pure API service with no template rendering, (2) Need async performance for I/O-bound operations, (3) Want automatic OpenAPI documentation, (4) Type safety matters. I'd consider rebuilding the Nexus caching proxy in FastAPI since it's pure API with async caching needs. LawPrix and PGPulse benefit too much from Django's ORM and admin to justify switching.

---

## AI/ML Fundamentals

**Q67. What is RAG, and where does it break down?**

A: RAG (Retrieval-Augmented Generation) retrieves relevant documents from a vector store and passes them as context to an LLM, grounding responses in actual data rather than the model's training knowledge. It breaks down when: (1) Documents aren't well-indexed or embeddings are poor quality, (2) The question requires reasoning across multiple documents that don't rank high individually, (3) The LLM still hallucinates despite retrieved context. Fine-tuning would be better for domain-specific language or when you need the model to deeply understand your data's patterns, not just reference it.

**Q68. Explain embeddings, then explain what "512-dimensional" means.**

A: Embeddings are numerical representations of text (or images) that capture meaning. Similar items have similar embeddings. Think of it like coordinates - words with similar meaning are "close" in this space. 512-dimensional means each embedding is a list of 512 numbers. Each dimension captures some aspect of meaning - one dimension might encode "formality", another "technical-ness", another "sentiment". The model learns these dimensions during training. More dimensions = finer-grained meaning capture, but more storage and computation.

**Q69. Cosine similarity vs Euclidean distance for face embeddings?**

A: I use cosine similarity. For high-dimensional embeddings like 512-d face vectors, the magnitude of the vector can vary based on lighting, angle, or image quality, but the direction captures the identity. Cosine similarity measures the angle (direction) between vectors, ignoring magnitude. Euclidean distance is affected by both direction and magnitude, making it less robust for face recognition where lighting conditions vary. Two faces of the same person might have different magnitudes but similar directions.

**Q70. Explain Random Forest from first principles.**

A: A single decision tree makes decisions by asking questions (if feature X > threshold, go left). It's prone to overfitting - it memorizes training data. Random Forest creates many decision trees (e.g., 100), each trained on a random subset of data and random subset of features. At inference time, each tree "votes" on the prediction. The final answer is the majority vote (classification) or average (regression). This ensemble approach reduces overfitting because individual trees make different errors, and averaging cancels them out. It's like asking 100 doctors for a diagnosis instead of relying on one.

**Q71. Precision vs recall for your three ML components?**

A: **Case Routing**: Precision matters more. Routing a case to the wrong lawyer (false positive) is worse than a case waiting longer (false negative). You want high confidence in each match. **Meal Prediction**: Recall matters slightly more. Under-predicting means hungry residents (bad), over-predicting means some waste (acceptable). **Face Recognition**: Equal importance, but false accept (wrong person gets attendance) is more dangerous than false reject (resident retries). I'd optimize for precision (low false accept rate) with a threshold of **[state your threshold]**.

**Q72. What's overfitting? How would you detect it in meal prediction?**

A: Overfitting is when a model memorizes training data instead of learning patterns. In meal prediction, signs would be: (1) Training accuracy is high but predictions for new days are poor, (2) Model performs well on one branch but poorly on others (branch-specific memorization), (3) Predictions exactly match historical data rather than generalizing. Detection: split data by time (train on weeks 1-3, test on week 4), not randomly, to simulate real deployment.

**Q73. How would you A/B test a model change on a live platform?**

A: Route a small percentage (5-10%) of new cases to the new model while keeping the old model for the rest. Compare metrics: lawyer acceptance rate, time to assignment, client satisfaction. Run for statistically significant sample size before full rollout. In LawPrix context, I'd measure: does the new model's routing lead to faster case acceptance? Are fewer cases reassigned? The risk is a bad model could route cases poorly for that 5-10%, so I'd only A/B test changes that have passed offline evaluation.

---

## Pressure-Testing Your Resume Claims

**Q74. "Client onboarding time dropped" at Atodya. How much?**

A: **[Provide specific numbers. Example: "Onboarding time dropped from approximately 2 hours to 30 minutes per client - a 75% reduction. This was measured by timing the process before and after introducing reusable Django app structures and templates. The source was operational measurement during my internship."]**

**Q75. "UI development time reduced" at Meru Technosoft. What's the before-after?**

A: **[Provide specifics. Example: "UI development time for new pages dropped from roughly 3 days to 1.5 days - about 50% reduction. This was measured by comparing sprint velocity before and after the component library was introduced. The library covered 80% of common UI patterns."]**

**Q76. "Developer handoff friction dropped." How do you measure friction?**

A: **[Be specific. Example: "Friction was measured by the number of clarification questions developers asked during handoff and the time between design delivery and development start. Before: developers had 5-6 clarification questions per feature and started 1-2 days after handoff. After: questions dropped to 1-2, and development started same-day. The component library with consistent naming conventions made designs self-documenting."]**

**Q77. Landing pages at MCS Cargar - before and after numbers?**

A: **[Provide Lighthouse scores, load times. Example: "Before: Lighthouse performance score was 65, First Contentful Paint was 3.2s. After lazy loading and code splitting: performance score improved to 92, FCP dropped to 1.1s. These were measured using Google Lighthouse in a simulated 4G environment."]**

**Q78. "2+ years of internship and freelance experience." How do the years stack up?**

A: **[Be transparent about overlap. Example: "Meru Technosoft was May 2024 - July 2025 (14 months). MCS Cargar was Jan 2025 - May 2025 (5 months, overlapping with Meru). Atodya was May 2026 - Jul 2026 (3 months). Total unique months: approximately 22 months. The overlap was intentional - Meru was part-time allowing concurrent freelance work. The 2+ years claim reflects total calendar time, not necessarily full-time equivalent."]**

---

> **Honesty is your best strategy.** Interviewers respect "I haven't measured that formally, here's what I observed" more than a made-up number. Fill in the brackets with your real data before the interview.
