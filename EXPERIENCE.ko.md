# 경력 상세

이력서의 긴 버전입니다. 어떤 문제가 있었고, 무엇을 어떻게 판단했고, 어떻게 됐는지를 적었습니다.
English: [EXPERIENCE.md](EXPERIENCE.md) · 이력서 PDF: [Kim_Keonwoo_CV.pdf](Kim_Keonwoo_CV.pdf) · [LinkedIn](https://www.linkedin.com/in/keonwoo-kim-profile/) · [keonwookim98@gmail.com](mailto:keonwookim98@gmail.com)

---

## Polyfact | AI Software Engineer
**2024.08 ~ 2025.09 · 파리**

프랑스 하원과 상원, EU 의회는 매일 수백 시간을 생중계하고 수천 쪽의 문서를 낸다. 로펌과 로비 회사는 그중 고객에게 걸리는 것만 찾아 24시간 안에 보고서를 내야 했고, 그 일을 사람이 손으로 했다. Polyfact는 이 일을 자동화하고 실시간으로 만드는 B2B 플랫폼이다. 인턴으로 합류했는데 곧 시니어 엔지니어들이 떠났고, CEO에게서 그 일을 직접 넘겨받아 제품, 백엔드, AI 파이프라인, SvelteKit 프론트엔드를 맡았다. 이후 합류한 엔지니어와 인턴의 코드 리뷰와 교육도 맡았다.

`Python` `FastAPI` `TypeScript (Deno, Node)` `SvelteKit` `Supabase/Postgres` `Pinecone` `Turbopuffer` `OpenAI` `Deepgram` `LangChain/LangGraph/LangSmith` `Tiptap` `hls.js` `Sentry` `OpenTelemetry` `PostHog` `GCP`

**정치인 프로필 자동 생성 (합류 후 첫 프로젝트).** 정치인 한 명을 파악하려면 위키, 기사, 의회 활동, SNS를 사람이 전부 뒤져야 했다. 당시에는 지금처럼 쓸 만한 API가 없어서, 키워드로 구글을 검색해 상위 기사 100여 건과 위키피디아, 신뢰할 만한 페이지, 정치인의 Facebook과 LinkedIn 프로필까지 모아 신뢰도로 거르고 요약해 하나의 프로필로 만드는 파이프라인을 혼자 구현했다. 정치인이 수천 명이라 전부 미리 만들 수 없어서, 그 프로필에 처음 들어온 방문자가 생성을 트리거하게 했다. 처음에는 자료를 한 번에 모델에 넣다가 token 한도에 걸려 요청이 실패했는데, 요청을 쪼개 batch로 처리하고 결과를 합치도록 바꾸고 요청당 token 사용량을 기록해 batch 크기를 조절했다.

**정치 데이터 수집과 검색 파이프라인.** 의회 포털, 정치인 SNS, 언론, 위원회 PDF에서 매일 새 데이터가 쏟아진다. 소스마다 cron이 watermark로 새것만 가져오고, doc_id와 정치인 id, 법안 id, 기관, 날짜로 메타데이터를 통일하고, 긴 문서는 300~500 단어로 자르되 앞뒤를 겹치게 해서 자른 자리에서 문맥이 끊기거나 내용이 빠지지 않게 했다. Postgres에는 정확해야 하는 메타데이터와 표결을, vector DB에는 검색용 임베딩을 넣었다. 검색은 벡터와 키워드(BM25)를 동시에 돌려 rank fusion으로 합치고, 기관과 연도로 나눈 namespace와 메타데이터 필터로 범위를 먼저 좁힌다. 처음에는 Pinecone으로, 이후 수정안 챗의 retrieval은 Turbopuffer로 만들었다. Turbopuffer는 저장소 제약에 맞춰 French BM25 tokenizer를 쓰고, 긴 본문은 4KB 제한이 있는 filter용 attribute에서 빼고, 문서 하나가 실패해도 적재가 멈추지 않게 했다.

**고객 맞춤 모니터링.** 고객은 자기 주제에 걸리는 의회 활동만 받고 싶어 했다. 처음에는 키워드 매칭이라 "에너지"라는 단어만 겹쳐도 원자력 논의가 재생에너지 담당자에게 갔다. 고객의 관심 프로필과 의회 활동을 모두 임베딩해 의미로 매칭하도록 바꿨고, 그 위에서 맞춤 리포트와 주제별 자동 알림이 돌아갔다.

**수정안 페이지 (가장 오래 붙들고 있던 작업).** 법안 하나에 수정안이 수백 건씩 달리고, 로비 회사는 그중 자기 고객에게 걸리는 것을 찾아 의견을 내야 한다. 법안(dossier)별 수정안 목록과 검색, 필터를 만들고, 사용자가 그 페이지에 들어오면 서버가 ai-backend에 dossier를 넘겨 수정안마다 AI 리포트를 미리 만들어 DB에 쌓게 했다. 사용자가 직접 쓴 리포트가 있으면 그것을 우선 보여 주고, export는 이미 만들어 둔 리포트를 DB에서 읽어 Word로 내보내되 아직 생성 전이면 끝날 때까지 요청을 대기시킨다. 수정안 polling은 GitHub Actions cron에서 GCP Scheduler로 옮겼고, 계속 커지던 리포트 생성 엔드포인트는 따로 리팩터링했다. 화면은 다 만들고도 피드백을 받아 갈아엎었고, 마지막에는 로비스트가 실제로 훑는 방식에 맞춰 엑셀처럼 보는 UI로 바꿨다. 이때 Videos, Biographies, Amendments, Documents, Agenda의 검색과 필터 UI도 하나로 통일했다.

**라이브 의회 워크스페이스.** 4시간짜리 본회의를 끝까지 볼 사람은 없다. 영상 페이지를 만들어 왼쪽에 라이브 영상과 AI가 나눈 챕터, 가운데에 Deepgram 실시간 transcript, 오른쪽에 챗과 메모장을 뒀다. 실시간으로 누가 말하는지 화자를 인식해 어느 정치인인지 찾아내고, 앞서 만들어 둔 프로필과 연결해 발언자 정보를 바로 볼 수 있게 했다. 말하는 문장이 하이라이트되고 화면이 따라 스크롤되며, 문장을 클릭하거나 영상을 옮기면 서로 맞춰 움직인다. 챗 답변을 옮겨 적는 Tiptap 메모장과 transcript, 답변 export도 만들었다. 라이브 플레이어가 자꾸 죽던 문제는 우리가 과하게 조정한 hls.js 설정이 원인이어서, 기본값으로 되돌려 해결했다.

**고객사별 맞춤 라이브 챗 context.** 회의 도중의 질문에는 방금 일어난 일을, 그것도 묻는 고객사의 관심사 기준으로 답해야 한다. transcript, 현재 화자, 논의 중인 수정안 같은 라이브 상태에 고객사마다의 기본 정보와 memory(관심사)를 더해 context로 모으는 pipeline을 조사하고 설계해 구현했다. 팀에 구조를 설명해 제품에 들어가게 했다.

**수정안 실시간 상태와 영상 매칭 (Deno KV, Queue).** 하원의 Eliasse는 본회의 중 수정안마다 처리 상태를 실시간으로 보여 준다. 이 업데이트를 영상 transcript의 해당 시점에 붙이는 매칭을 만들어 영상 페이지에 넣었고, 이후 이를 돌리던 백엔드를 이어받았다. cron이 고정 주기로 무조건 돌며 같은 회의를 반복 처리해 upsert 충돌을 냈고, 두 번째 queue listener가 첫 번째를 조용히 덮어쓰고 있었다. 스캐너가 1분마다 새 라이브만 찾아 queue에 넣고, CAS lock으로 두 worker가 같은 회의를 잡지 않고, 모든 queue 메시지를 라우터 하나로 받도록 다시 설계했다.

**앱 전역 검색.** 찾으려는 게 정치인인지 문서인지 수정안인지 모르는 채로 검색하는 경우가 많다. 페이지마다 흩어져 있던 검색을 맥의 Spotlight처럼 입력창 하나로 모으고, 키워드가 정확히 겹치지 않아도 의미로 걸리도록 임베딩 검색을 함께 돌려 정치인, 문서, 수정안, 영상, 일정을 한 번에 찾게 했다.

**AI 답변 속도.** 문서에 질문할 때마다 백엔드가 전체를 처음부터 다시 처리해 한 번에 10~15초가 걸렸다. 답을 streaming으로 내보내고 같은 질문은 캐시에서 바로 주도록 바꿔 2초 안쪽으로 줄였다.

**AI 백엔드와 스택 마이그레이션.** 빈 저장소에서 AI 백엔드를 만들었고, 이후 회사가 낸 모든 AI 기능이 그 위에서 돌았다. 첫 버전부터 요청마다 token 비용을 기록했다. 생성 요청이 줄줄이 대기하던 원인은 동기 라이브러리가 막고 있던 I/O였고, async로 다시 짜 동시에 처리되게 했다. 100페이지 문서 구조화는 진행률을 보여 주는 streaming으로 바꿨다. 스택은 필요할 때마다 옮겼다. 프론트와 타입이 어긋나던 Python FastAPI를 Deno로, 저장소 열 개 가까이 흩어져 같은 함수가 세 군데 있던 마이크로서비스를 Turborepo 모노레포로, 최신 AI와 vector DB 라이브러리가 Deno를 지원하지 않아 다시 Node로 옮겼다. 옮길 때는 기존 버전과 새 버전을 나란히 띄우고 엔드포인트 단위로 트래픽을 넘기며 문제가 생기면 바로 되돌릴 수 있게 해서 서비스가 멈추지 않게 했다. Svelte도 5가 안정 버전이 되자 바로 올렸다.

**Observability.** 고객이 "느리다"고 말해야 문제를 알던 상태였다. 백엔드 전반의 error handling을 정리해 공용 logger를 통해 Sentry로 보내고, 호출부를 고치지 않고 4개 서비스에 적용했다. 어떤 에러가 누구에게 몇 번 났는지 Discord로 바로 알림이 오게 했고, OpenTelemetry로 프론트부터 API, DB, 모델 호출까지 구간별 시간을 쟀고, PostHog로 사용 흐름을 봤다. 상원 생중계 중 transcript 지연이 튀었을 때 chunking 병목을 짚어 그날 고쳤다. OpenTelemetry는 비디오 pipeline에 혼자 계측했는데, Deno Deploy가 custom runtime flag를 받지 않아 프로덕션에서 돌릴 수 없었다. 그래서 Sentry를 넓혀 공백을 메우고 새 작업은 GCP로 옮겼으며, 이 결정이 결국 Node 마이그레이션으로 이어졌다.

**새 도구 도입과 판단.** LangChain, LangGraph, LangSmith를 챗과 리포트 agent에 도입하고 운영을 맡았다. 새 모델과 API 기능은 벤치마크가 아니라 우리 서비스 출력으로 검증하고 보통 출시 며칠 안에 반영했다. 웹 검색은 당시 OpenAI API로는 출처가 제대로 나오지 않아서, Tavily를 따로 붙여 파이프라인에 넣었다.

**팀 공통 개발 기준.** 장애 하나에 로그 하나만 남기는 error-handling 아키텍처(RFC 7807), 팀용 observability 가이드, AI 코딩 도구가 같은 컨벤션을 따르게 하는 rules 파일을 만들었다. 공용 패키지는 owner별 TODO로 나눠 작은 PR로 병렬 마이그레이션했고, 팀 PR 대부분을 리뷰했다.

그 밖에 일정(Agenda), 법안과 수정안 문서(Textes), 맞춤 알림, 설정 페이지도 처음부터 만들었다.

> 6개월 인턴 후 계속 근무 제안을 받아 이어서 근무. CEO 추천서(2024.12): "top 1%" in conscientiousness, "his issues, PR comments and technical notes are the most organized and consistent", "I would bet on him."

---

## Belage | Freelance Full-Stack Engineer
**2025.10 ~ 2026.09 · 파리**

여러 브랜드를 운영하는 사진 스튜디오에 웹사이트조차 없었다. SNS 계정으로 알리고 예약은 구글 폼으로 받아 일일이 옮겨 적었고, 상담과 결제, 정산도 손으로 했다. 웹사이트부터 백오피스까지 전부 새로 만들고, 반복되던 일을 뒷단에서 자동화했다. 혼자 만들고 운영한다.

`Next.js` `TypeScript` `Supabase/Postgres (RLS, PL/pgSQL, pg_cron)` `SumUp (online + card-present)` `Google Calendar API` `Vercel` `pgTAP`

**브랜드별 고객 사이트, 하나의 코드베이스.** 브랜드마다 도메인을 두고 한 코드베이스에서 배포한다. 카테고리별 포트폴리오, 패키지와 날짜 선택부터 참고 사진 업로드와 결제까지 이어지는 예약 과정, 예약 조회, 고객이 후기와 별점, 사진을 남기는 리뷰 페이지, 6개 언어를 갖췄다. 예약 캘린더는 날짜별 남은 촬영 시간을 분 단위로 받아서, 패키지를 바꾸면 추가 요청 없이 바로 다시 거르고, 270일치를 요청 한 번으로 가져온다.

**4.5MB에서 막히던 사진 업로드.** 참고 사진을 서버에서 검증(magic bytes, 해상도, rate limit)하도록 바꾸자 Vercel의 요청 한도 4.5MB에 걸려, 6.16MB 휴대폰 사진이 프로덕션에서 413으로 막히는 걸 확인했다. 브라우저에서 긴 변 2,400px, 3MB 이하로 줄여 올리게 해 한도 자체를 없앴고, HEIC는 서버에서 변환한다.

**모든 브랜드를 담는 Postgres schema 하나.** 고객 사이트와 스튜디오의 스케줄, 직원 권한, 인보이스를 한 schema에서 운영한다. 예약과 결제 상태 전이, 인보이싱은 RLS와 server-side role check 뒤의 단일 트랜잭션 PL/pgSQL RPC로 돌려서, 중간에 실패해도 반쯤 처리된 예약이 남지 않는다.

**맞아떨어지는 결제 (SumUp).** 온라인 결제와 매장 카드 결제를 하나의 atomic settlement path로 합쳤다. webhook은 idempotent하게 받고 provider에 다시 조회해 검증하며, 브라우저가 놓친 결제는 reconciliation cron이 찾아내고, 환불 credit note는 provider가 정산을 확인한 뒤에만 발행한다.

**프랑스 법에 맞춘 인보이싱.** facture/avoir 번호는 빠지거나 겹치면 안 되는 법정 요건(CGI art. 242 nonies A)이라, atomic counter 뒤에 별도 DB constraint를 한 겹 더 뒀다. 정산되면 PDF를 만들어 메일로 보내고, 국세청 현금영수증 export와 pg_cron 리마인더도 돌린다.

**스튜디오의 하루를 자동화.** 카카오톡, WhatsApp, Instagram, 이메일로 흩어진 상담을 진행 단계마다 자동 메시지로 보내고, 사진작가별 Google Calendar를 양방향으로 sync하고, 6개 언어를 한 소스에서 관리한다. 가격, 템플릿, 브랜드까지 백오피스에서 바꿀 수 있어 개발자 없이도 스튜디오가 시스템을 넓혀 간다.

---

## 직접 만들고 운영하는 제품

### [Bonjour Admin](https://bonjouradmin.com) | AI Software Engineer & Founder
**2026.02 ~ 현재 · 유료 구독자**

프랑스에 사는 외국인은 사람마다 상황이 다르고, 틀린 답 하나가 체류 자격을 흔들 수 있다. 범용 챗봇은 일반론으로 답한다. Polyfact에서 만들던 RAG 파이프라인을 더 최신 구성으로 혼자 다시 설계해, 묻는 사람의 상황에 맞춰 프랑스 법령과 공식 안내만 근거로 수치와 조항까지 확인해 6개 언어로 답하는 제품으로 냈다. 만들고 운영하고 수익화까지 혼자 한다.

`Next.js 16 / React 19` `TypeScript` `Supabase + pgvector` `OpenAI Responses API` `Cohere rerank` `Upstash Redis` `Polar.sh` `PostHog` `Vercel`

**Personalised retrieval.** 같은 질문도 사람마다 답이 달라서, 프로필(국적, 지역, 체류 자격, 가족, 직업)이 tag와 RRF boost로 retrieval을 조정하고 적용할 법 체계를 고른다. 사용자가 외국인 체류 포털 데이터를 연결하면, reverse-engineering으로 매핑한 110개 이상 endpoint 기준으로 파싱해 답에 쓴다.

**공식 소스만 근거로.** Légifrance 법전 6개는 매일, 정부 소스 15개는 매주 다시 수집한다. 웹 검색은 55개 도메인 whitelist 안에서만 하고, 법정 수치는 버전 관리해 ground truth로 넣는다.

**근거가 약할 때를 아는 retrieval.** retrieval gate → semantic cache → full-text와 dual embedding(pgvector) weighted RRF → Cohere rerank → quality gate 순서로 돈다. 근거가 약하면 quality gate가 검색 결과를 버리고, agent가 tool로 공식 소스를 다시 찾는다. agent는 함수 10개로 tool-calling을 최대 3라운드 하고, 초안은 병렬 verifier 3개(CRAG, legal-reference existence, citation entailment)가 검사한다. LLM이 쓴 SQL은 공개 fact 테이블 4개만 읽을 수 있는 anonymous role로 실행된다.

**비용과 latency를 재고 나서 줄이기.** stage별 cost telemetry를 붙여 보니 verification이 비용의 80%, web pre-search가 법률 답변 latency의 49%였다. pre-search를 cache가 오래됐을 때만 돌게 바꿔 검색이 18초에서 0.9초가 됐고, follow-up 질문에서는 retrieval gate가 검색의 60~70%를 건너뛴다.

**법이 바뀌어도 따라가는 eval.** 100건 이상 eval set과 LLM-as-judge 지표 6개를 만들고, judge 점수를 교차 확인하려고 모델 없이 계산하는 deterministic metrics를 함께 뒀다. 법령 수치 케이스는 정답을 버전 관리되는 facts 테이블에서 실행 시점에 읽어서, 법이 바뀌면 테스트도 고칠 필요 없이 따라간다. 매주 CI에서 돌고 회귀가 나오면 실패한다.

**코드 밖의 일까지 혼자.** pipeline을 만들기 전에 fake-door test로 수요부터 확인했다. 기능별 quota가 있는 Polar 구독, programmatic SEO 페이지 1,000개 이상, PostHog funnel, GDPR와 EU AI Act에 맞춘 약관 3개 언어, INPI 상표 등록까지 했다. 챗 외에 마감일 리마인더, 494문항 귀화 시험 트레이너, 문서 분석, 커뮤니티 경험담이 있다.

### SonnanAI | AI Software Engineer
**2026.01 ~ 현재 · 파리**

파리의 한 교회는 예배마다 자원봉사자들이 붙어 한국어 설교를 프랑스어로 통역했다. 사람이 매주 감당하던 그 일을 전부 자동화해, 지금은 통역자 없이 실시간 자막과 음성이 나간다.

`Python` `Flask (SSE)` `Silero VAD` `OpenAI transcribe + local Whisper` `Qwen2.5-7B LoRA` `Ollama` `pytest`

**예배 중에 멈추지 않는 pipeline.** 현장 네트워크가 자주 끊겨서 모든 단계에 fallback을 뒀다. VAD로 발화를 자르고, cloud STT가 안 되면 local Whisper로 넘기고, 번역은 weekly cache → local LoRA → cloud 3단으로 간다. 8초 wall clock 안에서 request hedging을 하고, cloud STT에는 circuit breaker를 달았고, 네트워크 없이 도는 offline 경로도 있다.

**직접 만든 corpus로 Qwen2.5-7B LoRA fine-tuning.** 3년치 한불 설교 아카이브를 정규화하고 문장 단위로 맞춰 약 8K 쌍을 만들었고, Ollama로 로컬 서빙한다. 라이브 세션마다 맥락이 담긴 재학습 쌍이 쌓이는 distillation flywheel을 두었고, train과 eval 데이터가 섞이지 않게 막았다.

**실측으로 정한 guard.** STT는 무음에서 없는 문장을 지어낸다. 그래서 실제 예배에서 잰 logprob 분포로 hallucination gate를 정했다(발화 -0.12 이상, 환각 -1.3 미만). 매주 미리 번역해 두는 cache가 발화의 18.5%를 흡수한다.

---

## École 42 Paris | IT Architecture Expert, RNCP 7 (석사 수준)
**2023.10 ~ 2026.10 · 파리**

Data and database architecture 전공. 강의도 교수도 없이 프로젝트와 동료 평가로만 진행하는 과정으로 28개 프로젝트를 했다. 라이브러리로 덮지 않고 밑바닥부터 구현한 뒤, 맞는지 기준값과 대조하는 방식으로 공부했다.

### AI / ML

**[numpy-only MLP](https://github.com/keonwoo98/Multilayer-Perceptron).** backpropagation을 손으로 유도해 구현하고 Adam과 L2를 얹었다. 직접 짠 gradient를 수치 미분과 대조하는 gradient check로 검증했다(최대 오차 1.72e-10). 정확도는 96%인데 test BCE가 0.33이었다. 어려운 샘플 몇 개를 거의 확신하며 틀리고 있었기 때문이라, mini-batch, L2, early stopping으로 과신을 막아 test BCE를 0.0499(정확도 98.59%)로 낮췄다.

**[EEG brain-computer interface](https://github.com/keonwoo98/Total-perspective-vortex).** 뇌파에서 어느 쪽 손을 움직이려 했는지 읽어 내는 과제. CSP를 두 클래스 공분산의 generalised eigenvalue problem으로 구현하고, 고유값 분해도 Jacobi 회전으로 직접 짜 SciPy와 1e-15까지 일치시켰다. CSP를 cross-validation fold 밖에서 학습하면 test 정보가 새어 점수가 1.0까지 부풀었고, fold마다 다시 학습하게 고쳐 정직한 0.844를 얻었다.

**[CNN 잎 병해 분류기](https://github.com/keonwoo98/Leaffliction) (PyTorch).** conv block 4개로 직접 설계했다. 첫 학습에서 validation 100%가 나온 게 단서였다. 증강을 먼저 하고 나눠서 같은 잎의 변형이 train과 validation 양쪽에 들어가 있었다. 먼저 나누고 학습 중에만 증강하도록 바꿔 held-out 99.79%를 얻었고, fine-tuned EfficientNet-B0(99.86%)와 같은 수준이다.

**[Q-learning snake](https://github.com/keonwoo98/Learn2Slither).** 뱀 머리에서 본 네 방향 시야만 입력으로 주는 제약이 있었다. 거리까지 넣으면 보드 크기에 묶이니까 방향별로 무엇이 있는지만 state로 인코딩했고, 그 덕분에 학습에 쓰지 않은 크기의 보드에서도 그대로 동작한다. 그 외에 linear/logistic regression도 라이브러리 없이 구현했다.

### Systems, Security, Blockchain

**[x86 kernel 직접 구현](https://github.com/keonwoo98/KFS-3) (C, NASM).** libc 없이 recursive page directory 기반 higher-half paging, Multiboot memory map으로 만든 frame allocator, kmalloc/vmalloc heap을 구현했다. paging을 켜는 순간에도 코드는 아직 저메모리에서 돌기 때문에, bootstrap page directory에 identity와 higher-half를 함께 매핑했다. 디버거도 stdout도 없어서, QEMU monitor로 VGA 메모리를 덤프해 화면에 뭐가 찍혔는지 확인하는 regression test를 만들었다.

**[Rust 오목 엔진](https://github.com/keonwoo98/Gomoku).** Negamax에 null-move pruning, transposition table, Lazy SMP를 얹어 한 수 500ms 안에 depth 10~17을 탐색한다. 게임 로그에서 탐색 깊이가 4까지 무너진 국면이 있어 그 자리를 회귀 테스트로 고정하고, 원인을 하나씩 분리해 고쳤다.

**Solidity 컨트랙트 2개.** OpenZeppelin 없이 BEP-20 토큰을 직접 구현하고, mint와 소유권 이전을 2-of-3 multisig 승인 뒤에 뒀다. 서명자를 빼다가 남은 인원이 필요 승인 수보다 적어지면 아무것도 실행할 수 없게 잠기는데, 배포한 컨트랙트는 고칠 수 없어서 제거 전에 검사하도록 막았다. 두 번째는 NFT인데 이미지 파일을 어디에도 올리지 않고 SVG와 메타데이터를 컨트랙트 안에서 만들어 완전히 온체인에 뒀다. 문자열을 이어 붙이다 stack too deep에 걸려 생성 함수를 셋으로 쪼개고 Yul 파이프라인으로 컴파일했다.

**그 외.** 서버가 피스 데이터를 보내지 않고 seed 하나만 내려 주면 모든 클라이언트가 같은 순서를 재현하는 멀티플레이어 테트리스(테스트 커버리지 94.58%, 엔진 100%), Vagrant와 K3s로 클러스터를 세우고 ArgoCD로 GitOps 동기화까지 붙인 인프라 과제(2인 팀), OWASP Top 10 기반 취약점을 직접 뚫고 방어까지 문서화한 웹 보안 과제(2인 팀), Flutter 앱 8개.

## École 42 Seoul
**2021.03 ~ 2023.05 · 서울**

**[42 해커톤 대상](https://github.com/keonwoo98/42_Eduthon) (과학기술정보통신부 장관상).** 3인 팀을 이끌고 순수 C로 BMP 이미지를 다루는 교육용 과제를 만들었다. 과제 명세와 reference implementation, 자동 채점기까지 한 세트로 설계했고, 42 Amsterdam, Brussels, Paris에서 발표했다.

**[C++98로 만든 미니 nginx](https://github.com/keonwoo98/webserv) (3인 팀).** nginx 문법의 설정 파일을 읽어 가상 호스트와 location을 구성하고, kqueue 논블로킹 이벤트 루프 하나로 static file, 업로드, CGI를 처리하는 HTTP/1.1 서버. 설정 파서와 서버 메인 로직을 맡았다. 잘못된 설정은 서버가 뜨기 전에 걸리도록 로딩 시점에 문법을 먼저 검증했고, server에서 location으로 설정이 상속되게 해 호출부가 값의 출처를 몰라도 되게 했다. 요청이 헤더 중간에서 끊겨도 상태를 들고 다음 이벤트에서 이어 파싱한다.

**멀티플레이어 Pong 웹 게임 (NestJS + React).** 42 OAuth와 2FA 로그인, socket.io 실시간 채팅(채널과 DM, 차단), 랭킹과 업적까지 붙인 실시간 대전 Pong. 42 학생들이 실제로 들어와 썼다. 채팅과 친구 도메인을 맡아 소켓 이벤트와 화면 상태가 한 방향으로 흐르게 정리했다. 이후 같은 서비스를 마이크로서비스로 나눠 다시 만든 팀 프로젝트에서는 Next.js 클라이언트를 맡았다.

**그 외.** libc, bash subset shell, dining philosophers, raycasting FPS 엔진(C), STL container(C++98), hardened Debian VM과 Docker stack.

---

## 기술

- **Open source**: CUBRID (2021): 스토리지 엔진의 Double Write Buffer 병목을 프로파일링해 최적화 패치를 upstream에 기여, 코어 팀에 결과 발표.
- **Proficient**: TypeScript, Python, C/C++, SQL (PostgreSQL)
- **Experienced with**: Rust, Deno, Solidity, Dart/Flutter, x86 assembly
- **AI systems**: RAG (hybrid retrieval, RRF, reranking), agentic tool loops, CRAG, evals (LLM-as-judge), LLM cost and latency engineering, output sandboxing, LoRA fine-tuning, local serving (Ollama), speech pipelines (VAD, STT, TTS)
- **Backend & Web**: Next.js, FastAPI, Flask, SvelteKit, Node/Deno, Supabase/Postgres (RLS, PL/pgSQL), Redis, SSE/WebSocket
- **Infrastructure**: GCP Cloud Run, Vercel, Docker, K3s + ArgoCD, GitHub Actions, Sentry, OpenTelemetry, PostHog
- **언어·병역**: 한국어(모국어) · 영어(업무) · 프랑스어(기초) · 군필(육군 헌병, 2019.01 ~ 2020.08)
