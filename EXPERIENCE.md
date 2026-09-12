# Experience in detail

The long version of my CV: what the problem was, what I decided, and how it turned out.
한국어: [EXPERIENCE.ko.md](EXPERIENCE.ko.md) · CV (PDF): [Kim_Keonwoo_CV.pdf](Kim_Keonwoo_CV.pdf) · [LinkedIn](https://www.linkedin.com/in/keonwoo-kim-profile/) · [keonwookim98@gmail.com](mailto:keonwookim98@gmail.com)

---

## Polyfact — AI Software Engineer
**Aug 2024 – Sep 2025 · Paris**

The French National Assembly, the Senate, and the EU Parliament stream hundreds of hours and publish thousands of pages every day. Law and lobbying firms have to find the specific items touching their clients and write actionable briefs within 24 hours—a process previously handled entirely by hand. Polyfact automates that workflow and makes it real-time. I joined as an intern, but following senior engineering departures shortly after, the CEO handed over full ownership of the product, backend, AI pipelines, and SvelteKit frontend architecture. Later, I spearheaded code reviews, technical onboarding, and training for incoming engineers and interns.

`Python` `FastAPI` `TypeScript (Deno, Node)` `SvelteKit` `Supabase/Postgres` `Pinecone` `Turbopuffer` `OpenAI` `Deepgram` `LangChain/LangGraph/LangSmith` `Tiptap` `hls.js` `Sentry` `OpenTelemetry` `PostHog` `GCP`

**Automated Politician Profile Generation.** Synthesizing a single politician's background required manually auditing Wikipedia, press articles, parliamentary records, and social media. Lacking usable unified APIs at the time, I engineered an automated pipeline from scratch: searching Google by keyword to collect ~100 top articles alongside Wikipedia, trusted pages, and the politician's Facebook and LinkedIn profiles, filtering them for credibility, and summarizing the material into a single profile. Because pre-generating profiles for thousands of politicians was impractical, I designed an on-demand generation pipeline triggered asynchronously upon a visitor's first page load. Initial implementations passed raw source material directly to the LLM, triggering token limit failures. I resolved this by refactoring the ingestion into batched requests, merging the outputs, and logging per-request token usage to dynamically tune batch sizes.

**Political Data Ingestion and Search Pipeline.** Parliamentary portals, politician social feeds, press releases, and committee PDFs yield daily data streams. I set up per-source watermark crons to ingest delta changes exclusively, normalizing metadata schemas across doc_id, politician ID, bill ID, institution, and date. Long-form documents are partitioned into 300–500 word overlapping chunks to preserve semantic context across chunk boundaries. Deterministic data requiring strict accuracy (such as voting records and metadata) is stored in Postgres, while vector embeddings reside in a vector store. Search queries execute vector similarity and BM25 keyword searches concurrently, combining results via Reciprocal Rank Fusion (RRF) and narrowing the search space using institution/year namespaces and metadata pre-filtering. Initially backed by Pinecone, I migrated retrieval for the amendment chat to Turbopuffer to leverage its French BM25 tokenizer and custom storage bounds—excluding full document bodies from the 4KB filterable attribute limits and ensuring single-document ingestion failures never halt the indexing pipeline.

**Client-Specific Monitoring.** Clients required targeted intelligence relevant strictly to their domain. Naive keyword matching caused noise—such as routing nuclear energy debates to a renewable-energy client simply because both contained the word "energy". I embedded both client interest profiles and political activity records to perform semantic matching on intent, powering personalized report generation and automated topic alerts.

**The Amendments Workspace.** Individual bills frequently accumulate hundreds of amendments, requiring lobbying firms to quickly filter relevant items and issue responses. I built per-bill amendment pages featuring real-time search and filters. When a user accesses a bill, the server triggers the AI backend to pre-generate and cache AI reports for every amendment in the background. User-edited reports take precedence over generated ones, and document exports read cached DB reports to assemble Word documents—holding export requests until background generation finishes if still in-flight. I migrated amendment polling from GitHub Actions crons to GCP Scheduler and modularized the growing report-generation endpoint. Following user feedback, I redesigned the frontend into a high-density grid/table view matching how lobbyists manually scan amendments, unifying search and filter UIs across videos, biographies, amendments, documents, and agenda pages.

**Live Parliamentary Interactive Workspace.** The National Assembly, Senate, and EU Parliament stream concurrently for hours every day. To solve the multi-stream monitoring challenge, I built a unified video workspace: featuring live video and AI-generated chaptering on the left, a Deepgram real-time transcript in the middle, and chat plus a notebook on the right. Real-time speaker identification maps active speakers to politician profiles, enabling instant profile popups, synchronized transcript highlighting, auto-scrolling, and timestamp seeking. I also implemented a Tiptap notebook allowing users to save chat answers into notes and export transcripts. Live HLS streams frequently experience segment delays or drops; naive buffer and retry adjustments caused warning accumulation until fatal errors crashed the player entirely. I overhauled stream recovery based on hls.js guidelines: classifying errors to handle level, fragment, and key loading failures via `startLoad`, escalating to `recoverMediaError` on every third attempt, and resetting consecutive error counters after 30 seconds of stability to prevent crash loops. For unrecoverable 404 missing segments, the system trips a circuit breaker after 5 retries to notify the user of incomplete recording, aborts in-flight requests, and gracefully cleans up player resources. Buffers, timeouts, and retry budgets were recalibrated specifically for live streaming rather than VOD.

**Live, Client-Aware Context Injection.** Answering questions mid-sitting requires contextual awareness of both real-time stream events and the specific client asking. I researched, designed, and deployed a context injection pipeline that synthesizes live stream state (running transcript, current speaker, active amendments) with client-specific profiles and interest memories, establishing it as a team-wide standard.

**Real-Time Amendment Status Tracking (Deno KV, Queues).** The French Assembly's Eliasse feed broadcasts live amendment statuses during sittings. I engineered matching logic to link these real-time status updates directly to timestamped video transcripts, then took ownership of the underlying backend. The original system ran fixed-interval crons that reprocessed identical sittings, causing upsert conflicts and queue listener race conditions. I re-architected the system around Deno KV and queue workers: a 1-minute delta scanner enqueues new live sessions exclusively, CAS locks prevent multiple workers from contending for the same sitting, and a single router processes all queue messages to ensure concurrency control and load distribution.

**App-Wide Unified Search.** Users frequently search without knowing whether a resource is categorized as a politician, document, amendment, or video. I consolidated fragmented page searches into a single Spotlight-style global search box, executing keyword and vector embedding searches in parallel. This enables semantic discovery across politicians, documents, amendments, videos, and agendas even when search terms do not match verbatim.

**Sub-Second Answer Latency Optimization.** Document Q&A initially reprocessed entire documents from scratch, leading to 10–15 second latency per response. I eliminated duplicate data fetches, converted independent execution stages into non-blocking parallel async routines, and re-budgeted token allocations. Incorporating response streaming and caching brought end-to-end response times down to under 2 seconds.

**AI Backend Architecture and Stack Migrations.** I engineered Polyfact's dedicated AI backend from an empty repository, serving as the foundational platform for all subsequent AI capabilities. Per-request token cost logging was built in from version 1. When generation requests queued up, I identified blocking I/O within synchronous libraries as the bottleneck and refactored the pipeline to be fully async to expand concurrency, while shifting 100-page document structuring to a streaming progress architecture. I managed tech stack migrations as scale demanded: moving from Python FastAPI to Deno to fix frontend type drift, consolidating ~10 fragmented microservices into a Turborepo monorepo to eliminate duplicate code, and eventually migrating to Node.js when cutting-edge AI and vector DB libraries lacked Deno runtime support. Each migration was executed via dual-run side-by-side deployments, shifting traffic endpoint-by-endpoint with immediate rollback capabilities to guarantee zero service downtime. The frontend was upgraded to Svelte 5 as soon as it reached production stability.

**Observability Infrastructure.** To shift away from reactive debugging based on user complaints, I overhauled backend error handling to route logs through a shared logger to Sentry, reaching 4 microservices without call-site code changes and alerting the team on Discord. I instrumented OpenTelemetry to profile stage latency across frontend, API, database, and LLM calls, and used PostHog to analyze user traffic flows. When transcript latency spiked during a Senate livestream, tracing identified a chunking bottleneck that was patched the same day. After discovering Deno Deploy runtime limitations with custom OTel flags, I expanded Sentry's coverage, migrated new workloads to GCP, and led the technical decision to transition to Node.js.

**Engineering Standards and Tooling.** I introduced and owned LangChain, LangGraph, and LangSmith across chat and reporting agents, evaluating new LLMs and API features on our internal output datasets to ship updates within days of release. To address missing citation URLs in OpenAI's API at the time, I integrated Tavily Search directly into the retrieval pipeline. I established team-wide standards including RFC 7807 compliant structured error responses, an observability guide, and AI coding rules files, while leading parallel refactoring of shared packages via per-owner TODOs and reviewing the majority of team PRs.

I also built the agenda workspace, bills and text documents pages, custom alerts engine, and user settings pages from scratch.

> Offered a full-time position following a 6-month internship. CEO Recommendation (Dec 2024): "top 1%" in conscientiousness; "his issues, PR comments and technical notes are the most organized and consistent"; "I would bet on him."

---

## Belage — Freelance Full-Stack Engineer
**Oct 2025 – Sep 2026 · Paris**

A multi-brand photo studio operated without a web presence, taking bookings via Google Forms, manually transcribing data, and handling consultations, payments, and invoicing on paper. I built and operate the entire technical stack single-handedly—from customer-facing sites to back-office management systems—automating all underlying operations.

`Next.js` `TypeScript` `Supabase/Postgres (RLS, PL/pgSQL, pg_cron)` `SumUp (online + card-present)` `Google Calendar API` `Vercel` `pgTAP`

**Multi-Brand Tenant Web Infrastructure.** Deployed domain-isolated client portals for each brand from a single unified codebase. Built category portfolio galleries, booking flows with package and date selection, reference asset uploads, payment processing, booking lookup, customer reviews with star ratings and photos, and 6-language i18n support. The booking calendar dynamically calculates remaining shooting minutes per date, re-filtering instantly upon package switches without secondary API calls, and fetches a 9-month booking window in a single optimized request.

**Resolving 4.5 MB Payload Constraints.** Adding server-side reference image validation (magic bytes, dimensions, rate limiting) hit Vercel's 4.5 MB request payload cap, causing 6.16 MB mobile phone uploads to fail with HTTP 413 errors in production. I resolved this by implementing client-side browser canvas resizing (constraining long edges to 2,400px and files under 3MB) prior to upload, while processing HEIC formats via asynchronous server conversion.

**Single Postgres Schema Architecture.** Unified customer portals, studio scheduling, staff access roles, and invoicing under a single database schema. Encapsulated booking state transitions, payment updates, and invoice generation inside single-transaction PL/pgSQL RPCs behind RLS and server-side role checks to ensure transactional atomicity and prevent orphaned booking states.

**Payment Reconciliation.** Merged online and in-studio card payments into a single atomic settlement path. Implemented idempotent webhooks verified via provider re-queries, background reconciliation crons to catch dropped browser sessions, and credit note issuance conditioned strictly on confirmed provider settlements.

**Legal Invoicing Compliance.** Enforced strict compliance with French legal Facture/Avoir numbering rules (CGI art. 242 nonies A)—prohibiting gaps or duplicate sequences—by pairing atomic counters with database-level uniqueness constraints. Automated settlement PDF generation and email dispatch, Korean NTS cash receipt exports, and pg_cron-based payment reminders.

**Automated Studio Operations.** Integrated fragmented consultations across KakaoTalk, WhatsApp, Instagram, and email into stage-triggered automated messaging flows, built bi-directional Google Calendar sync per photographer, and centralized 6-language translations. Configured no-code back-office management for pricing, templates, and branding, allowing studio staff to expand operations independently.

---

## Products I build and run

### [Bonjour Admin](https://bonjouradmin.com) — AI Software Engineer & Founder
**Feb 2026 – Present · Paying subscribers**

Foreigners in France face highly individual administrative requirements where incorrect guidance can compromise residence permits. General chatbots fail with generic responses. Building on the RAG architectures I developed at Polyfact, I designed and launched a commercial product that delivers citation-backed answers in 6 languages, grounded strictly in French statutes and official guidance with verified figures and legal references. I handle end-to-end product development, infrastructure, operations, and monetization independently.

`Next.js 16 / React 19` `TypeScript` `Supabase + pgvector` `OpenAI Responses API` `Cohere rerank` `Upstash Redis` `Polar.sh` `PostHog` `Vercel`

**Personalized Retrieval Engine.** Designed profile-aware retrieval matching user context (nationality, region, visa status, family, profession) to apply tag filters, RRF score boosts, and dynamic legal framework selection. Reverse-engineered and integrated parsers mapping 110+ endpoints from official immigration portals to incorporate user-linked portal data into answer generation.

**Ground Truth Data Pipeline.** Configured daily automated syncing of 6 Légifrance legal codes, weekly recrawling of 15 government portals, web search restricted to a 55-domain whitelist, and a version-controlled legal database for statutory numbers and thresholds.

**Multi-Stage Retrieval and Verification.** Built a multi-tier pipeline: Retrieval Gate → Semantic Cache → Weighted RRF over full-text and dual embeddings (pgvector) → Cohere Rerank → Quality Gate. When retrieval confidence falls short, the Quality Gate rejects initial results, triggering agentic tool searches over official sources. The agent executes up to 3 tool-calling rounds across 10 functions, validating draft answers via 3 parallel verifiers (CRAG, legal-reference existence, citation entailment). Model-generated SQL queries execute under sandboxed anonymous database roles restricted to 4 public fact tables.

**Telemetry-Driven Performance Tuning.** Instrumented stage-level cost telemetry, revealing that verification accounted for 80% of total spend and web pre-search consumed 49% of legal answer latency. Restricting pre-search execution to stale cache states cut search latency from 18s to 0.9s, while the Retrieval Gate bypasses 60–70% of unnecessary searches on follow-up questions.

**Continuous Eval Pipeline for Statutory Changes.** Established an evaluation suite of 100+ cases evaluated via 6 LLM-as-judge metrics alongside non-LLM deterministic metrics. Test cases for statutory figures dynamically read expected ground truth from versioned fact tables at runtime, ensuring test suites automatically adapt to legislative updates without code modifications. Runs weekly in CI, failing builds on regression.

**End-to-End Business Execution.** Verified market demand via fake-door tests prior to pipeline development. Built Polar subscription infrastructure with per-feature quotas, generated 1,000+ programmatic SEO pages, configured PostHog conversion funnels, authored GDPR and EU AI Act compliant terms across 3 languages, and registered an INPI trademark. Surrounding the core chat, I developed deadline reminders, a 494-question citizenship exam trainer, document analysis tools, and community feature modules.

### SonnanAI — AI Software Engineer
**Jan 2026 – Present · Paris**

Automated Korean-to-French real-time speech translation and subtitle generation for live church events in Paris, replacing manual volunteer interpretation with a fully automated live audio pipeline.

`Python` `Flask (SSE)` `Silero VAD` `OpenAI transcribe + local Whisper` `Qwen2.5-7B LoRA` `Ollama` `pytest`

**High-Availability Live Audio Pipeline.** Engineered a fault-tolerant 3-tier fallback architecture resilient to local network drops and cloud API outages. Implemented Silero VAD segmentation, automated failover from Cloud STT to Local Whisper, and a 3-tier translation layer (Weekly Cache → Local LoRA → Cloud). Enforced an 8-second wall-clock limit via request hedging, added cloud STT circuit breakers, and built a fully offline execution path.

**Domain LoRA Fine-Tuning and Distillation.** Normalized and sentence-aligned 3 years of bilingual audio archives into ~8K parallel training pairs to fine-tune Qwen2.5-7B LoRA, served locally via Ollama. Constructed an automated distillation flywheel accumulating session context into retraining datasets while maintaining strict isolation between training and evaluation splits.

**Empirical Logprob Guarding.** Filtered STT hallucination in silent audio segments using logprob distributions measured from live sessions (speech > -0.12, hallucinations < -1.3). Across 23 live sessions, weekly pre-translation caching absorbed ~18% of total speech utterances (ranging from 3% to 52% depending on session topic).

---

## École 42 Paris — IT Architecture Expert, RNCP 7 (Master's level)
**Oct 2023 – Oct 2026 · Paris**

Specialized in Data and Database Architecture. Completed 28 peer-reviewed computer science projects emphasizing bottom-up implementation without external libraries.

### AI / ML

**[numpy-only MLP](https://github.com/keonwoo98/Multilayer-Perceptron).** Derived and implemented backpropagation algorithms from scratch using NumPy, incorporating Adam optimization and L2 regularization. Validated gradient calculations against numerical differentiation via gradient checking (max error 1.72e-10). Diagnosed an overconfidence issue on hard samples (96% accuracy with a high test BCE of 0.33) and corrected it using mini-batching, L2 regularization, and early stopping to achieve a test BCE of 0.0499 (98.59% accuracy).

**[EEG brain-computer interface](https://github.com/keonwoo98/Total-perspective-vortex).** Built a motor imagery classification model from EEG signals. Formulated Common Spatial Pattern (CSP) as a generalized eigenvalue problem over class covariance matrices, implementing a custom Jacobi eigensolver matching SciPy output to 1e-15 precision. Identified data leakage caused by fitting CSP outside cross-validation folds (which inflated validation scores to 1.0) and corrected the pipeline to refit CSP within each fold, securing an honest score of 0.844.

**[CNN leaf-disease classifier](https://github.com/keonwoo98/Leaffliction) (PyTorch).** Designed a custom 4-block CNN architecture. Investigated an initial 100% validation accuracy anomaly, discovering data leakage caused by applying data augmentation prior to dataset splitting. Corrected the pipeline to split datasets prior to augmentation, achieving 99.79% held-out test accuracy (matching a fine-tuned EfficientNet-B0 at 99.86%).

**[Q-learning snake](https://github.com/keonwoo98/Learn2Slither).** Implemented a Q-learning agent restricted to 4-direction local vision relative to the snake's head. Encoded relative directional state rather than explicit coordinates, enabling the trained agent to generalize across unobserved board dimensions. Also implemented linear and logistic regression models from scratch.

### Systems, security, blockchain

**[x86 kernel from scratch](https://github.com/keonwoo98/KFS-3) (C, NASM).** Built an x86 kernel featuring higher-half paging via recursive page directories, a Multiboot memory map frame allocator, and kmalloc/vmalloc heap management without standard libraries. Dual-mapped identity and higher-half memory in the bootstrap directory to maintain compatibility when enabling paging in low memory. Lacking a debugger or stdout, constructed regression test suites by dumping VGA memory via the QEMU monitor.

**[Rust Gomoku engine](https://github.com/keonwoo98/Gomoku).** Developed a Negamax search engine with null-move pruning, transposition tables, and Lazy SMP, achieving search depths of 10–17 under 500ms per move. Isolated and resolved a search depth collapse issue (where depth dropped to 4) by capturing problematic game log states as regression test cases.

**Solidity Contracts.** Authored a custom BEP-20 token contract without OpenZeppelin, placing minting and ownership transfers behind a 2-of-3 multisig authorization mechanism with safeguard validation checks against unexecutable signer removals. Built a fully on-chain NFT contract that generates SVG graphics and metadata dynamically within the contract; resolved "stack too deep" compilation errors by splitting generation logic into 3 functions and compiling through Yul pipelines.

**Additional Systems Work.** Built a multiplayer Tetris platform where the server broadcasts random seeds to synchronize identical piece sequences across clients (94.58% test coverage, 100% engine coverage); engineered VM infrastructure via Vagrant, K3s clusters, and ArgoCD GitOps pipelines; performed OWASP Top 10 web vulnerability exploitation and defense documentation; developed 8 Flutter applications.

## École 42 Seoul
**Mar 2021 – May 2023 · Seoul**

**[Grand Prize, 42 Hackathon](https://github.com/keonwoo98/42_Eduthon) (Minister of Science and ICT Award).** Led a 3-person team to create a C-based educational project centered on BMP image manipulation—delivering project specifications, a reference implementation, and an autograder. Presented the package across 42 Amsterdam, Brussels, and Paris campuses.

**[A small nginx in C++98](https://github.com/keonwoo98/webserv).** Built an HTTP/1.1 web server parsing Nginx-style configurations into virtual hosts and locations, processing static files, uploads, and CGI asynchronously on a single non-blocking kqueue event loop. Designed configuration parsing with load-time syntax validation, automatic server-to-location setting inheritance, and state-machine request parsing capable of resuming incomplete HTTP header buffers.

**Multiplayer Pong on the web (NestJS + React).** Developed a real-time Pong platform featuring 42 OAuth, 2FA, Socket.io chat (channels, DMs, blocking), and leaderboards. Architected unidirectional data flows between socket events and UI state.

**Additional Projects.** Re-implemented Libc functions, built a Bash-subset shell, solved Dining Philosophers concurrency problems, built a C raycasting FPS engine, implemented C++98 STL containers, and configured hardened Debian VMs with Docker stacks.

---

## Skills

- **Proficient:** TypeScript, Python, C / C++, SQL (PostgreSQL)
- **Experienced with:** Rust, Deno, Solidity, Dart / Flutter, x86 assembly
- **AI systems:** RAG (hybrid retrieval, RRF, reranking), agentic tool loops, CRAG, evals (LLM-as-judge), LLM cost and latency engineering, output sandboxing, LoRA fine-tuning, local serving (Ollama), speech pipelines (VAD, STT, TTS)
- **Backend & Web:** Next.js, FastAPI, Flask, SvelteKit, Node / Deno, Supabase/Postgres (RLS, PL/pgSQL), Redis, SSE / WebSocket
- **Infrastructure:** GCP Cloud Run, Vercel, Docker, K3s + ArgoCD, GitHub Actions, Sentry, OpenTelemetry, PostHog
- **Languages:** Korean (native) · English (professional) · French (basic)
- **Open source:** CUBRID (2021): Profiled Double Write Buffer bottlenecks in the storage engine, contributed upstream optimization patches, and presented findings to the core engineering team.

