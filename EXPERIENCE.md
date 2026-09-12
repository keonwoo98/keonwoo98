# Experience in detail

The long version of my CV: what the problem was, what I decided, and how it turned out.
한국어: [EXPERIENCE.ko.md](EXPERIENCE.ko.md) · CV (PDF): [Kim_Keonwoo_CV.pdf](Kim_Keonwoo_CV.pdf) · [LinkedIn](https://www.linkedin.com/in/keonwoo-kim-profile/) · [keonwookim98@gmail.com](mailto:keonwookim98@gmail.com)

---

## Polyfact — AI Software Engineer
**Aug 2024 – Sep 2025 · Paris**

The French National Assembly, the Senate and the EU Parliament stream hundreds of hours and publish thousands of pages every day. Law and lobbying firms have to find the few items that touch their clients and write a brief within 24 hours, and they were doing it by hand. Polyfact automates that work and makes it real time. I joined as an intern. The senior engineers left soon after, and the CEO handed their work to me: the product, the backend, the AI pipeline and the SvelteKit frontend. Later I reviewed and trained the engineers and interns who joined after me.

`Python` `FastAPI` `TypeScript (Deno, Node)` `SvelteKit` `Supabase/Postgres` `Pinecone` `Turbopuffer` `OpenAI` `Deepgram` `LangChain/LangGraph/LangSmith` `Tiptap` `hls.js` `Sentry` `OpenTelemetry` `PostHog` `GCP`

**Politician profiles, generated (my first project there).** Understanding one politician meant reading Wikipedia, the press, parliamentary activity and social media by hand. There were no usable APIs for this at the time, so I built the pipeline alone: search Google by keyword, pull the top hundred or so articles along with Wikipedia, trusted pages and the politician's Facebook and LinkedIn profiles, filter them for credibility, then summarise everything into one profile. With thousands of politicians, generating them all up front was impossible, so the first visitor to a profile triggers its generation. Sending all that source material to the model at once blew past token limits and requests failed, so I split them into batches, merged the results, and logged token use per request to tune batch size.

**Ingestion and search over political data.** Parliamentary portals, politicians' social media, the press and committee PDFs produce new material every day. A cron per source uses a watermark to fetch only what is new; metadata is normalised (doc_id, politician and bill ids, institution, date); long documents are cut into 300 to 500 word chunks that overlap, so nothing is lost or left without context at the seam. Postgres holds what has to be exact, such as metadata and voting records, and the vector store holds the embeddings. Search runs vector and BM25 keyword search together and merges them with rank fusion, and namespaces by institution and year plus metadata filters narrow the ground first. Pinecone came first; retrieval for the amendments chat was then built on Turbopuffer, around the store's limits: a French BM25 tokenizer, long text kept out of the 4KB filterable attributes, and one failed document never stopping an ingest.

**Monitoring tuned to each client.** Clients wanted only the parliamentary activity that touched their topics. Keyword matching meant a renewable-energy client got nuclear debates because both said "energy". I embedded both the client's interest profile and the political activities and matched on meaning instead, and personalised reports and topic alerts ran on top of it.

**The amendments page, the longest job I had there.** A single bill collects hundreds of amendments, and a lobbying firm has to find the ones that touch its client and respond. I built the amendments page per bill with search and filters, and made the server hand the bill to the AI backend when a user opens it, so a report is generated for every amendment and stored. A user's own report takes precedence over the generated one, and export reads the stored reports and produces a Word document, holding the request until generation finishes if it is still running. Amendment polling moved from a GitHub Actions cron to GCP Scheduler, and the report-generation endpoint, which had grown far too large, was refactored out. The page was finished once, sent back after feedback and rebuilt, and ended up as a spreadsheet-style view because that is how lobbyists actually scan amendments. Along the way I unified the search and filter UI across the videos, biographies, amendments, documents and agenda pages.

**A live parliament workspace.** Nobody watches four hours of plenary. I built the video page: the live stream and AI-generated chapters on the left, a live Deepgram transcript in the middle, chat and a notebook on the right. Speakers are identified while the session runs, resolved to the politician speaking, and linked to the profiles generated earlier, so their record is one click away. The sentence being spoken is highlighted, the view scrolls with it, and clicking a sentence or moving the video keeps both in step. I also built the Tiptap notebook clients wrote into from the chat, and export for transcripts and answers. When the live player kept dying, the cause was our own over-tuned hls.js configuration; going back to the defaults fixed it.

**Live, client-aware context in the chat.** A question asked mid-sitting has to be answered about what just happened, and for the firm asking. I researched, designed and built the pipeline that assembles that context: the live state (transcript so far, current speaker, amendments under debate) plus each client firm's profile and its memory of what it cares about. I walked the team through it until it shipped.

**Amendment status matched to the video (Deno KV, queues).** During a sitting, the Assembly's Eliasse feed shows the status of every amendment. I built the matching that places those updates at the right moment of the video transcript and shipped it on the video page, then took over the backend that ran it. Its crons fired on a fixed schedule regardless, re-processed the same sittings and hit upsert conflicts, and a second queue listener was silently overwriting the first. I rebuilt it on Deno KV state and a queue: a scanner looks for new lives every minute and enqueues only those, CAS locks keep two workers off the same sitting, and one router receives every queue message.

**Search across the whole app.** People often search without knowing whether what they want is a politician, a document, an amendment or a video. I pulled the searches scattered across pages into a single box, Spotlight style, and ran embedding search alongside keywords so a query finds the right thing even when the words do not match exactly, across politicians, documents, amendments, videos and the agenda.

**Answers that arrive in seconds.** Asking a question about a document reprocessed the whole thing from scratch, so each answer took 10 to 15 seconds. I streamed the answer as it was generated and cached repeated questions, which brought it under 2 seconds.

**The AI backend, and the migrations it went through.** I built the AI backend from an empty repo, and every AI feature the company shipped afterwards ran on it. Token cost was logged per request from the first version. When generation requests started queueing, the cause was blocking I/O inside synchronous libraries; I rewrote the path async so they ran concurrently, and moved 100-page document structuring to streaming with live progress. The stack moved whenever it had to: Python FastAPI to Deno because types kept diverging from the frontend, close to ten scattered microservice repos into a Turborepo monorepo where the same function existed in three places, then back to Node because the newest AI and vector-database libraries did not support Deno. Each move ran the old and new versions side by side, shifted traffic endpoint by endpoint and could be rolled back immediately, so the service never went down. Svelte went to 5 as soon as it was stable.

**Observability, so the client is not the alarm.** Problems used to surface when a client said "it's slow". I reworked error handling across the backend and sent it to Sentry through the shared logger, which reached four services with no call-site changes, with Discord alerts telling us what broke, for whom and how often. OpenTelemetry timed each segment from frontend to API to database to model call, and PostHog showed how the product was actually used. When transcript latency spiked during a Senate livestream, tracing pointed at a chunking bottleneck and it was patched the same day. I instrumented OpenTelemetry on the video pipeline alone, but Deno Deploy would not accept the custom runtime flags it needed, so I widened Sentry to cover the gap and moved new work to GCP, and that decision is what led to the Node migration.

**New tooling, adopted and judged.** I introduced LangChain, LangGraph and LangSmith for the chat and report agents and then owned them. New models and API capabilities were tested on our own outputs rather than benchmarks, usually within days of release. For web search, the OpenAI API of the time did not return usable sources, so I brought in Tavily and wired it into the pipeline myself.

**Standards the team coded against.** An error-handling architecture (RFC 7807, one log per failure), an observability guide for the team, a rules file so AI-assisted code followed the same conventions, and a per-owner TODO rollout so a shared package migrated in parallel through small PRs. I reviewed most of the team's PRs.

Also built from scratch: the agenda page, the bills and documents pages, custom alerts and settings.

> Offered to stay on after the six-month internship. CEO reference, Dec 2024: "top 1%" in conscientiousness; "his issues, PR comments and technical notes are the most organized and consistent"; "I would bet on him."

---

## Belage — Freelance Full-Stack Engineer
**Oct 2025 – Sep 2026 · Paris**

A multi-brand photo studio had no website at all. It advertised on social accounts, took bookings through a Google Form and copied them out by hand, and ran consultations, payments and invoicing manually. I built everything, from the customer sites to the back office, and automated the repetitive work behind them. I run it alone.

`Next.js` `TypeScript` `Supabase/Postgres (RLS, PL/pgSQL, pg_cron)` `SumUp (online + card-present)` `Google Calendar API` `Vercel` `pgTAP`

**Customer sites for every brand, one codebase.** Each brand has its own domain, deployed from the same codebase: a portfolio by category, a booking flow from package and date through reference photos to payment, booking lookup, a reviews page where clients leave ratings, comments and photos, six languages. The booking calendar returns the shooting minutes left on each date, so switching packages re-filters instantly with no second request, and one query covers the whole 270-day window.

**Uploads that broke at 4.5 MB.** Routing reference photos through the server for validation (magic bytes, dimensions, rate limit) put them under Vercel's 4.5 MB request cap, and a 6.16 MB phone photo came back 413 in production. Photos are now resized in the browser (2,400 px, under 3 MB) before upload, which removes the ceiling; HEIC is converted on the server.

**One Postgres schema for every brand:** the customer sites plus the studio's scheduling, staff roles and invoices. Booking and payment transitions and invoicing run as single-transaction PL/pgSQL RPCs behind RLS and server-side role checks, so a failure halfway never leaves half a booking.

**Payments that reconcile.** Online and in-studio card payments share one atomic settlement path. Webhooks are idempotent and re-verified against the provider, a reconciliation cron catches payments the browser lost, and credit notes are issued only after the provider confirms settlement.

**Invoicing by French law.** Facture and avoir numbers may have no gaps or duplicates (CGI art. 242 nonies A), so numbering runs on an atomic counter with an independent database constraint behind it. PDFs are generated and emailed on settlement, Korean NTS cash receipts are exported, and reminders run on pg_cron.

**The studio's day, automated.** Consultations arriving on KakaoTalk, WhatsApp, Instagram and email go out as stage-triggered messages, each photographer's Google Calendar syncs both ways, and six languages come from one source. Prices, templates and even brands are editable from the back office, so the studio extends the system without a developer.

---

## Products I build and run

### [Bonjour Admin](https://bonjouradmin.com) — AI Software Engineer & Founder
**Feb 2026 – Present · paying subscribers**

Every foreigner in France is a different case, and one wrong answer can put a residence permit at risk. General chatbots give general answers. I rebuilt the retrieval pipeline I had grown at Polyfact on a newer, stronger stack and shipped it as a product: answers for the person asking, in six languages, from French law and official guidance only, with figures and references checked. Built, run and monetised alone.

`Next.js 16 / React 19` `TypeScript` `Supabase + pgvector` `OpenAI Responses API` `Cohere rerank` `Upstash Redis` `Polar.sh` `PostHog` `Vercel`

**Personalised retrieval.** The same question has different answers for different people, so a profile (nationality, region, permit status, family, work) steers retrieval through tags and RRF boosts and picks the legal framework that applies. Users can attach their own immigration-portal data, parsed against 110+ endpoints I mapped by reverse-engineering.

**Official sources only.** Six legal codes synced daily from Légifrance, fifteen government sources recrawled weekly, web search limited to a 55-domain whitelist, statutory figures versioned and injected as ground truth.

**Retrieval that knows when it is weak.** Retrieval gate, semantic cache, weighted RRF over full-text and two embeddings (pgvector), Cohere rerank, then a quality gate: when the evidence is weak, the gate drops it and the agent searches official sources with its tools instead. The agent works in up to three tool-calling rounds over ten functions, and three verifiers check the draft in parallel (CRAG, legal-reference existence, citation entailment). SQL written by the model runs as an anonymous role that can read four public fact tables and nothing else.

**Cost and latency, measured, then cut.** Per-stage cost telemetry showed verification at 80% of spend and web pre-search at 49% of a legal answer's latency. Pre-search now runs only when the cache is stale, which took search from 18 s to 0.9 s, and the retrieval gate skips 60–70% of searches on follow-up questions.

**Evals that follow the law when it changes.** A 100+ case eval set with six LLM-as-judge metrics, and deterministic metrics that use no model, to cross-check the judge. Legal-figure cases read their expected answer from the versioned facts table at run time, so when a figure changes in law the tests follow without edits. Runs weekly in CI and fails on regression.

**Everything beyond the code, alone.** A fake-door test before building the pipeline, Polar subscriptions with per-feature quotas, 1,000+ programmatic SEO pages, PostHog funnels, GDPR- and EU AI Act-compliant terms in three languages, an INPI-registered trademark. Around the chat: deadline reminders, a 494-question citizenship-exam trainer, document analysis, community experiences.

### SonnanAI — AI Software Engineer
**Jan 2026 – Present · Paris**

A Paris church relied on volunteers to interpret Korean sermons into French, service after service. I automated that work end to end: subtitles and speech now run live without an interpreter in the loop.

`Python` `Flask (SSE)` `Silero VAD` `OpenAI transcribe + local Whisper` `Qwen2.5-7B LoRA` `Ollama` `pytest`

**A pipeline that doesn't stop mid-service.** The venue's network drops often, so every stage has a fallback: VAD segmentation, cloud STT with local Whisper behind it, translation in three tiers (weekly cache → local LoRA → cloud), request hedging inside an 8 s wall clock, a circuit breaker on cloud STT, and a fully offline path.

**Qwen2.5-7B LoRA on a corpus I built.** Three years of bilingual sermon archives normalised and sentence-aligned into ~8K pairs, served locally through Ollama. Every live session turns into context-aware retraining pairs (a distillation flywheel), with train/eval leakage blocked.

**Guards set from measurements.** STT invents sentences in silence, so the hallucination gate uses logprob distributions measured in real services (speech above -0.12, phantoms below -1.3). A weekly pre-translation cache absorbs 18.5% of utterances.

---

## École 42 Paris — IT Architecture Expert, RNCP 7 (Master's level)
**Oct 2023 – Oct 2026 · Paris**

Data and database architecture option. No lectures and no professors: projects and peer evaluation only, 28 of them. The method was always the same, build it from the bottom instead of covering it with a library, then check the result against a reference.

### AI / ML

**[numpy-only MLP](https://github.com/keonwoo98/Multilayer-Perceptron).** Backpropagation derived by hand, with Adam and L2, and checked by gradient checking against numerical derivatives (max error 1.72e-10). Accuracy was 96% but test BCE sat at 0.33: a few hard samples were being misclassified with near-certainty. Mini-batches, L2 and early stopping brought test BCE to 0.0499 (98.59% accuracy).

**[EEG brain-computer interface](https://github.com/keonwoo98/Total-perspective-vortex).** Reading from brain signals which hand someone intended to move. CSP written as a generalised eigenvalue problem over two class covariances, with my own Jacobi eigensolver matching SciPy to 1e-15. Fitting CSP outside the cross-validation folds leaked test data and inflated the score to 1.0; refitting it inside each fold gave the honest 0.844.

**[CNN leaf-disease classifier](https://github.com/keonwoo98/Leaffliction) (PyTorch).** A four-block CNN I designed. The first run scored 100% on validation, which was the clue: augmenting before splitting had put variants of the same leaf on both sides. Splitting first and augmenting only during training gave 99.79% held-out, on par with a fine-tuned EfficientNet-B0 (99.86%).

**[Q-learning snake](https://github.com/keonwoo98/Learn2Slither).** The rules allowed the agent only what the snake sees in four directions from its head. Encoding distance would have tied it to one board size, so the state records only what is in each direction, and the trained agent plays board sizes it never saw. Linear and logistic regression were written from scratch too.

### Systems, security, blockchain

**[x86 kernel from scratch](https://github.com/keonwoo98/KFS-3) (C, NASM).** Higher-half paging with a recursive page directory, a frame allocator built from the Multiboot memory map, kmalloc and vmalloc heaps, no libc. Paging is switched on while the code still runs from low memory, so the bootstrap directory maps both. With no debugger and no stdout, regression tests dump VGA memory through the QEMU monitor and check what is on screen.

**[Rust Gomoku engine](https://github.com/keonwoo98/Gomoku).** Negamax with null-move pruning, a transposition table and Lazy SMP; depth 10–17 under 500 ms. A position from a game log where the search collapsed to depth 4 became a regression test, and its causes were separated and fixed one by one.

**Two Solidity contracts.** A BEP-20 token written without OpenZeppelin, with mint and ownership transfer behind 2-of-3 multisig approval. Removing signers can leave fewer of them than the approvals required, which locks the contract forever and cannot be patched after deployment, so removal is checked first. The second is an NFT that stores no image anywhere: the SVG and the metadata are generated inside the contract, fully on chain. Concatenating those strings hit "stack too deep", so generation was split into three functions and compiled through the Yul pipeline.

**Also:** multiplayer Tetris where the server broadcasts only a seed and every client reproduces the same piece sequence (94.58% test coverage, 100% on the engine); an infrastructure project provisioning VMs with Vagrant, a K3s cluster and GitOps sync through ArgoCD (team of two); a web security project exploiting OWASP Top 10 vulnerabilities by hand and documenting the defences (team of two); eight Flutter apps.

## École 42 Seoul
**Mar 2021 – May 2023 · Seoul**

**[Grand Prize, 42 Hackathon](https://github.com/keonwoo98/42_Eduthon) (Minister of Science and ICT Award).** I led a team of three to build a teaching project in pure C around BMP images: the assignment itself, a reference implementation and an autograder, as one package. Presented at 42 Amsterdam, Brussels and Paris.

**[A small nginx in C++98](https://github.com/keonwoo98/webserv) (team of three).** An HTTP/1.1 server that reads an nginx-style configuration file into virtual hosts and locations and serves static files, uploads and CGI on a single non-blocking kqueue event loop. I owned the config parser and the server's main logic: syntax is validated at load time so a bad config fails before the server starts, settings inherit from server to location so callers never need to know where a value came from, and a request cut in the middle of its headers keeps its state and resumes on the next event.

**Multiplayer Pong on the web (NestJS + React).** A real-time Pong game with 42 OAuth and 2FA login, socket.io chat with channels, DMs and blocking, rankings and achievements. Students at 42 actually played it. I owned the chat and friends domains and made socket events and screen state flow in one direction. A later team project split the same service into microservices, where I owned the Next.js client.

**Also:** libc, a bash-subset shell, dining philosophers and a raycasting FPS engine (C); STL containers (C++98); a hardened Debian VM and Docker stack.

---

## Skills

- **Proficient:** TypeScript, Python, C / C++, SQL (PostgreSQL)
- **Experienced with:** Rust, Deno, Solidity, Dart / Flutter, x86 assembly
- **AI systems:** RAG (hybrid retrieval, RRF, reranking), agentic tool loops, CRAG, evals (LLM-as-judge), LLM cost and latency engineering, output sandboxing, LoRA fine-tuning, local serving (Ollama), speech pipelines (VAD, STT, TTS)
- **Backend & Web:** Next.js, FastAPI, Flask, SvelteKit, Node / Deno, Supabase/Postgres (RLS, PL/pgSQL), Redis, SSE / WebSocket
- **Infrastructure:** GCP Cloud Run, Vercel, Docker, K3s + ArgoCD, GitHub Actions, Sentry, OpenTelemetry, PostHog
- **Languages:** Korean (native) · English (professional) · French (basic)
- **Open source:** CUBRID (2021): Double Write Buffer bottleneck profiled in the storage engine, optimisation patch contributed upstream, findings presented to the core team.
