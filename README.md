# Keonwoo Kim

AI engineer in Paris. I build LLM systems that have to be right, and run them in production.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-keonwoo--kim--profile-0A66C2?logo=linkedin&logoColor=white)](https://www.linkedin.com/in/keonwoo-kim-profile/)
[![CV](https://img.shields.io/badge/CV-PDF-444?logo=adobeacrobatreader&logoColor=white)](https://github.com/keonwoo98/keonwoo98/blob/main/Kim_Keonwoo_CV.pdf)
[![Bonjour Admin](https://img.shields.io/badge/bonjouradmin.com-live-1F6F5B)](https://bonjouradmin.com)
[![Email](https://img.shields.io/badge/keonwookim98%40gmail.com-D14836?logo=gmail&logoColor=white)](mailto:keonwookim98@gmail.com)

## Now

| | What it does | How it's built | Status |
|---|---|---|---|
| **[Bonjour Admin](https://bonjouradmin.com)** | Personalised, verified answers on French administration in six languages, from actual law | Hybrid RAG (pgvector, RRF, Cohere rerank) → agentic tool loop → three parallel verifiers · LLM-as-judge evals · per-stage cost telemetry · Next.js, Supabase, OpenAI Responses API | Live, paying subscribers. Founder, sole engineer |
| **SonnanAI** | Real-time Korean → French interpretation for a Paris church, live subtitles and speech | Silero VAD → cloud STT with local Whisper fallback → weekly cache → Qwen2.5-7B LoRA on Ollama → cloud → TTS · fully offline path · logprob hallucination gate | Live every Sunday since March 2026 |
| **Belage** | Booking, payments and French statutory invoicing for a Paris photo studio | Next.js · Supabase/Postgres (RLS, PL/pgSQL) · SumUp online + card-present · Google Calendar sync · six languages | In production, run alone |

<!-- Optional: a 10–15 s screen recording of Bonjour Admin answering a question. Save as assets/bonjour-admin.gif (keep under 5 MB) and uncomment:
<p align="center"><img src="assets/bonjour-admin.gif" width="760" alt="Bonjour Admin answering a question with citations"></p>
-->

## Before

**AI Software Engineer, Polyfact** (Paris policy-intelligence startup, 2024–2025). Built the LLM backend from an empty repo, introduced RAG (Pinecone, then Turbopuffer) and LangChain/LangGraph/LangSmith, owned monitoring across four services. Intern to permanent contract in six months.

## What you can read here

The products above are closed-source. The public repos are the École 42 track, written from scratch without ML frameworks.

| Repo | What | Result |
|---|---|---|
| [Multilayer-Perceptron](https://github.com/keonwoo98/Multilayer-Perceptron) | numpy-only MLP, hand-derived backpropagation, Adam, L2 | Gradient check agrees to 1.72e-10 |
| [Total-perspective-vortex](https://github.com/keonwoo98/Total-perspective-vortex) | EEG motor-imagery BCI, CSP written from scratch | Matches SciPy to 1e-15 |
| [Leaffliction](https://github.com/keonwoo98/Leaffliction) | CNN leaf-disease classifier, leak-safe augmentation | 99.79% held-out, on par with a fine-tuned EfficientNet-B0 |
| [Learn2Slither](https://github.com/keonwoo98/Learn2Slither) | Q-learning snake under partial observability | Generalises to board sizes it never saw |
| [Gomoku](https://github.com/keonwoo98/Gomoku) | Rust engine: Negamax, null-move pruning, transposition table, Lazy SMP | Depth 10–17 under 500 ms |
| [KFS-1](https://github.com/keonwoo98/KFS-1) → [KFS-3](https://github.com/keonwoo98/KFS-3) | x86 kernel in C and NASM: higher-half paging, recursive page directory, kmalloc/vmalloc | Regression-tested headlessly via the QEMU monitor |
| [webserv](https://github.com/keonwoo98/webserv) | HTTP/1.1 server in C++98 on a non-blocking kqueue event loop | 42 Seoul |
| [42_Eduthon](https://github.com/keonwoo98/42_Eduthon) | BMP image-processing teaching project in pure C, with reference implementation and autograder | Grand Prize, 42 Seoul Hackathon (Minister of Science and ICT Award) |

## Stack

![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![C](https://img.shields.io/badge/C%20%2F%20C%2B%2B-00599C?logo=c&logoColor=white)
![Rust](https://img.shields.io/badge/Rust-000?logo=rust&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL%20%2F%20pgvector-4169E1?logo=postgresql&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-000?logo=nextdotjs&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white)
![LangGraph](https://img.shields.io/badge/LangGraph-1C3C3C?logo=langchain&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama-000?logo=ollama&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![GCP](https://img.shields.io/badge/Google%20Cloud-4285F4?logo=googlecloud&logoColor=white)
