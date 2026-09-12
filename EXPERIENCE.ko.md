# 경력 상세

이력서의 긴 버전입니다. 어떤 문제가 있었고, 무엇을 어떻게 판단했고, 어떻게 됐는지를 적었습니다.
English: [EXPERIENCE.md](EXPERIENCE.md) · 이력서 PDF: [Kim_Keonwoo_CV.pdf](Kim_Kim_Keonwoo_CV.pdf) · [LinkedIn](https://www.linkedin.com/in/keonwoo-kim-profile/) · [keonwookim98@gmail.com](mailto:keonwookim98@gmail.com)

---

## Polyfact | AI Software Engineer
**2024.08 ~ 2025.09 · 파리**

프랑스 하원과 상원, EU 의회는 매일 수백 시간을 생중계하고 수천 쪽의 문서를 낸다. 로펌과 로비 회사는 그중 고객에게 걸리는 이슈만 찾아 24시간 안에 보고서를 내야 했고, 그 일은 전적으로 사람이 수작업으로 처리하고 있었다. Polyfact는 이 과정을 자동화하고 실시간화하는 B2B 플랫폼이다. 인턴으로 합류했으나 얼마 지나지 않아 시니어 엔지니어들이 이탈하면서, CEO로부터 제품 전체, 백엔드, AI 파이프라인, SvelteKit 프론트엔드 아키텍처 인프라 전반을 넘겨받아 전담했다. 이후 합류한 엔지니어와 인턴의 코드 리뷰 및 교육도 맡았다.

`Python` `FastAPI` `TypeScript (Deno, Node)` `SvelteKit` `Supabase/Postgres` `Pinecone` `Turbopuffer` `OpenAI` `Deepgram` `LangChain/LangGraph/LangSmith` `Tiptap` `hls.js` `Sentry` `OpenTelemetry` `PostHog` `GCP`

**정치인 프로필 자동 생성.** 정치인 한 명을 파악하려면 위키, 기사, 의회 활동, SNS를 사람이 전부 뒤져야 했다. 당시에는 바로 쓸 만한 통합 API가 없어, 키워드로 구글을 검색해 상위 기사 100여 건과 위키피디아, 신뢰할 수 있는 웹페이지, 정치인의 Facebook 및 LinkedIn 프로필까지 수집한 뒤 신뢰도로 정제·요약해 단일 프로필로 합성하는 파이프라인을 구축했다. 수천 명에 달하는 정치인을 미리 생성할 수 없었기에, 프로필에 처음 들어온 방문자가 생성을 트리거하도록 비동기 온디맨드 구조로 설계했다. 초기에는 수집 자료를 한 번에 LLM에 전달하다가 token 한도로 요청 실패가 발생했다. 이를 해결하기 위해 요청을 쪼개 batch 단위로 처리하고 결과를 합치도록 전환했으며, 요청당 token 사용량을 기록해 batch 크기를 동적으로 조절했다.

**정치 데이터 수집과 검색 파이프라인.** 의회 포털, 정치인 SNS, 언론사, 위원회 PDF 문서에서 매일 새로운 데이터가 쏟아진다. 소스마다 watermark cron을 두어 델타(변경분) 데이터만 수집하고, doc_id, 정치인 id, 법안 id, 기관, 날짜로 메타데이터를 규격화했다. 긴 문서는 300~500 단어로 잘라 오버랩 저장함으로써 자른 경계에서 문맥이 끊기거나 정보가 누락되지 않게 했다. Postgres에는 높은 데이터 정합성이 요구되는 메타데이터와 표결 내역을 배치하고, Vector DB에는 검색용 임베딩을 저장했다. 검색 시 벡터 유사도 검색과 키워드 검색(BM25)을 동시에 실행해 Rank Fusion(RRF)으로 결합하고, 기관과 연도로 구분한 namespace 및 메타데이터 필터로 검색 범위를 먼저 선별했다. 초기에는 Pinecone을 활용했으나, 수정안 챗의 retrieval 인프라는 Turbopuffer로 이관했다. Turbopuffer 저장소 제약에 맞춰 French BM25 tokenizer를 설정하고, filter용 attribute에서 4KB 제한이 걸리는 긴 본문을 제외했으며, 개별 문서 실패 시에도 인덱싱 적재가 중단되지 않는 처리 로직을 적용했다.

**고객 맞춤 모니터링.** 고객은 자신들의 사업 영역 및 관심사 영역의 의회 활동만 정확히 추적하길 원했다. 기존 키워드 매칭 방식 방식에서는 "에너지"라는 단어만 포함되어도 원자력 안건이 재생에너지 담당자에게 전달되는 한계가 있었다. 이를 해결하고자 고객 관심 프로필과 의회 활동 데이터를 모두 임베딩하여 의미 기반(Semantic Matching)으로 매칭되도록 전환했고, 이 아키텍처 위에서 맞춤형 리포트 및 주제별 자동 알림 파이프라인을 운영했다.

**수정안 페이지.** 법안 하나에 수정안이 수백 건씩 발의되며, 로비 회사는 그중 고객사와 관련된 항목을 신속히 선별해 의견을 제출해야 한다. 법안(dossier)별 수정안 목록, 검색, 필터 기능을 구현하고, 사용자가 페이지 진입 시 서버가 AI 백엔드로 법안 데이터를 넘겨 수정안별 AI 리포트를 비동기로 미리 생성·캐싱하도록 했다. 사용자가 직접 수정한 리포트가 있을 경우 이를 최우선 반환하고, export 시에는 DB에 축적된 리포트를 읽어 Word 파일로 내보내되 아직 생성 중인 요청은 완료될 때까지 대기 처리했다. 수정안 polling 인프라는 GitHub Actions cron에서 GCP Scheduler로 옮겼으며, 계속 커지던 리포트 생성 엔드포인트를 모듈화해 리팩터링했다. 또한 로비스트의 실제 문서 검토 방식에 맞춰 전체 뷰를 엑셀 형태의 Grid/Table UI로 재설계하여 적용했다. 이 과정에서 Videos, Biographies, Amendments, Documents, Agenda 등 흩어져 있던 검색 및 필터 UI 체계도 하나로 통합했다.

**라이브 의회 워크스페이스.** 하원, 상원, EU 의회가 같은 날 동시 다발적으로 생중계를 진행하여 통합 모니터링 화면이 필요했다. 라이브 영상, AI 기반 자동 챕터 분할, Deepgram 실시간 transcript, 챗 및 메모장을 단일 뷰로 통합한 워크스페이스를 개발했다. 실시간 화자 인식을 통해 발언 중인 정치인 프로필을 즉시 매칭하고, 발언 문장 하이라이트, 자동 스크롤, 문장 클릭 시 영상 타임스탬프 이동이 상호 연동되도록 구현했다. Tiptap 기반 메모장과 transcript 연동, 답변 export 기능도 구축했다. 라이브 HLS 스트리밍 특성상 세그먼트 누락이나 지연이 자주 발생했다. 단순 재시도 처리 방식은 경고가 누적되다가 fatal error 발생 시 플레이어 전체가 crash되는 문제가 있었다. 이를 개선하기 위해 hls.js 권장 가이드를 기반으로 복구 로직을 재설계했다. 에러 유형별로 구분하여 level, fragment, key 로딩 실패는 startLoad로 재접속하고 3회 연속 실패 시 recoverMediaError로 승격시켰으며, 30초간 에러가 없으면 연속 에러 카운터를 초기화하여 루프 로딩을 방지했다. 404 누락 세그먼트는 5회 재시도 후 한도를 초과하면 요청을 abort하고 플레이어를 안전하게 정리하도록 처리했다. 버퍼, 타임아웃, 재시도 예산 역시 VOD 기준이 아닌 라이브 스트리밍 환경에 맞게 재설정했다.

**고객사별 맞춤 라이브 챗 context.** 실시간 회의 진행 중 질의에는 생중계 방금 일어난 내용뿐만 아니라, 질의한 고객사의 사업 배경과 관심사를 기준으로 답변해야 한다. transcript, 현재 화자, 논의 중인 수정안 등 실시간 라이브 상태값에 고객사별 기본 정보 및 memory(관심사)를 결합하는 context 주입 pipeline을 조사·설계 및 구현하여 팀 내 표준으로 전파했다.

**수정안 실시간 상태와 영상 매칭 (Deno KV, Queue).** 프랑스 하원의 Eliasse 시스템은 본회의 수정안 처리 상태를 실시간 업데이트한다. 이 실시간 상태 변화를 영상 transcript의 발생 시점에 동기화하는 매칭 로직을 구현해 라이브 페이지에 적용했으며, 기존 백엔드 인프라를 인수해 재설계했다. 기존 방식은 cron이 고정 주기로 동일 회의를 중복 처리하여 upsert 충돌이 생기거나 queue listener가 이전 작업을 덮어쓰는 문제가 있었다. 이를 1분 주기 델타 스캐너 구조로 전환하여 새 라이브 세션만 queue에 전달하고, CAS lock을 도입해 두 worker가 동일 회의를 중복 점유하지 않도록 했으며, 단일 라우터를 통해 모든 queue 메시지를 중앙 제어하도록 재설계하여 동시성 제어 및 트래픽 부하를 최적화했다.

**앱 전역 검색.** 사용자가 정치인, 문서, 수정안 중 어떤 카테고리를 찾는지 지정하지 않아도 바로 검색할 수 있는 단일 검색 인터페이스를 구축했다. 맥의 Spotlight UX를 벤치마킹하여 검색창 하나로 모든 리소스를 통합하고, 키워드 검색과 임베딩 검색을 병행 처리해 정확한 키워드가 일치하지 않더라도 의미론적으로 관련 정치인, 문서, 수정안, 영상, 일정을 한 번에 탐색할 수 있게 했다.

**AI 답변 속도.** 문서 질의 요청 시마다 백엔드가 전체 프로세스를 처음부터 재처리하여 응답에 10~15초가 소요되었다. 중복 데이터를 호출하는 병목을 제거하고, 독립적인 단계는 병렬(Async Concurrency)로 전환했으며, 모델에 투입되는 토큰 예산 구조를 재조정했다. 여기에 답변 streaming 서빙과 캐시 레이어를 도입하여 end-to-end 응답 latency를 2초 안쪽으로 단축했다.

**AI 백엔드와 스택 마이그레이션.** 빈 저장소 상태에서 공통 AI 백엔드를 구축했으며, 이후 출시된 모든 AI 기능이 해당 인프라 위에서 구동되도록 기반을 만들었다. 최초 버전부터 요청별 token 비용 추적 모듈을 내장했다. 생성 요청 대기의 주요 원인이었던 동기(Synchronous) I/O 병목을 Non-blocking Async 구조로 전면 재작성하여 동시 요청 처리 성능을 개선했다. 100페이지 분량 문서의 구조화 작업은 실시간 진행률을 시각화하는 streaming 방식으로 전환했다. 기술 스택은 서비스 요구사항 변화에 맞춰 유연하게 이관했다. 프론트엔드와 타입 불일치가 발생하던 Python FastAPI를 Deno로 이관했고, 10여 개 저장소로 파편화되어 동일 함수가 중복 존재하던 microservice 구조를 Turborepo 기반 모노레포로 통합했다. 이후 최신 AI 및 Vector DB 라이브러리의 Deno 런타임 미지원 제약으로 인해 Node.js 환경으로 재이관했다. 이관 시에는 두 버전을 나란히 띄우는 Dual-run 인프라를 구성하고 엔드포인트 단위로 트래픽을 단계적 전환하여 무중단 이관을 달성했다. Svelte 5 릴리즈 시에도 안정화 즉시 프로덕션 업그레이드를 완료했다.

**Observability.** 고객의 장애 제보에 의존하던 수동적인 문제를 해결하기 위해 백엔드 전반의 에러 핸들링 구조를 체계화했다. 공용 logger를 구현하여 호출부 코드 수정 없이 4개 서비스 로그를 Sentry로 통합 집계했으며, 주요 에러 발생 시 Discord 알림 체계를 구축했다. OpenTelemetry를 도입해 프론트엔드부터 API, DB, 모델 호출 구간별 latency를 프로파일링하고, PostHog로 사용자 사용 패턴을 분석했다. 상원 생중계 중 발생한 transcript 지연 문제를 chunking 병목으로 원인 규명하여 당일 즉시 수정했다. Deno Deploy 런타임의 커스텀 플래그 미지원으로 인한 OTel 계측 제약을 확인한 후, Sentry 모니터링 범위를 확장하고 신규 워크로드를 GCP로 이관했으며, 이 판단을 기반으로 Node.js 마이그레이션 기술 의사결정을 주도했다.

**새 도구 도입과 판단.** LangChain, LangGraph, LangSmith를 챗 및 리포트 agent에 도입하여 운영을 전담했다. 신규 LLM 모델 및 API 기능은 벤치마크 점수에 의존하지 않고 내부 서비스 데이터 검증 파이프라인을 거쳐 릴리즈 며칠 내에 프로덕션에 신속 적용했다. 당시 OpenAI API의 검색 출처 미표기 한계를 보완하기 위해 Tavily 검색 API를 파이프라인에 직접 연동했다.

**팀 공통 개발 기준.** 단일 장애 발생 시 표준 에러 응답을 보장하는 구조(RFC 7807), 팀 observability 가이드, AI 코딩 도구용 규칙(rules) 파일 체계를 수립하고 전파했다. 공용 패키지 리팩터링 시 모듈 owner별 TODO 분할 방식을 도입해 소규모 PR 단위로 병렬 마이그레이션을 진행했으며, 팀 PR 전반에 대한 코드 리뷰를 전담했다.

그 외에도 일정(Agenda), 법안 및 수정안 문서(Textes), 맞춤 알림, 사용자 설정 페이지 등 핵심 도메인을 최초 구축했다.

> 6개월 인턴십 수료 후 계속 근무 제안을 받아 이어서 근무. CEO 추천서(2024.12): "top 1%" in conscientiousness, "his issues, PR comments and technical notes are the most organized and consistent", "I would bet on him."

---

## Belage | Freelance Full-Stack Engineer
**2025.10 ~ 2026.09 · 파리**

여러 브랜드를 운영하는 사진 스튜디오의 기존 수동 업무 프로세스(구글 폼 예약, 수작업 정산, 수동 메신저 상담)를 전면 자동화하기 위해 웹사이트부터 백오피스 관리 시스템까지 전 파이프라인을 구축 및 운영했다.

`Next.js` `TypeScript` `Supabase/Postgres (RLS, PL/pgSQL, pg_cron)` `SumUp (online + card-present)` `Google Calendar API` `Vercel` `pgTAP`

**브랜드별 고객 사이트, 하나의 코드베이스.** 단일 코드베이스 기반으로 브랜드별 도메인을 독립 배포하는 아키텍처를 구축했다. 카테고리별 포트폴리오, 패키지 및 날짜 선택, 참고 사진 업로드, 결제 연동, 예약 조회, 고객 후기/별점/사진 리뷰 기능, 6개 언어 다국어 지원을 구현했다. 예약 캘린더는 날짜별 잔여 촬영 시간을 분 단위로 계산하여 패키지 변경 시 추가 요청 없이 실시간 필터링하며, 9개월 치 예약 가능 슬롯을 단 1회 API 호출로 최적화해 가져온다.

**4.5MB에서 막히던 사진 업로드.** 참고 사진의 서버 검증(magic bytes, 해상도, rate limit) 로직 추가 후, Vercel 본문 용량 제한(4.5MB)으로 인해 6.16MB 상당의 모바일 사진 업로드가 HTTP 413 에러로 차단되는 문제를 확인했다. 클라이언트 브라우저 단에서 이미지 긴 변을 2,400px, 용량을 3MB 이하로 사전 압축해 업로드하도록 개선하여 Vercel 페이로드 한도 문제를 해결했으며, HEIC 포맷은 서버 비동기 변환 프로세스로 처리했다.

**모든 브랜드를 담는 Postgres schema 하나.** 전 브랜드 고객 사이트와 백오피스(스케줄, 직원 권한, 인보이스)를 단일 DB Schema로 통합 운영한다. 예약·결제 상태 전이 및 인보이스 생성 로직을 RLS 및 Server-side Role Check 검증 후 단일 트랜잭션 PL/pgSQL RPC로 처리하여, 프로세스 중단 시 불완전한 예약 데이터가 남지 않도록 데이터 정합성을 보장했다.

**결제 정합성 (SumUp).** 온라인 결제와 매장 카드 결제 파이프라인을 단일 Atomic Settlement Path로 통합했다. Idempotent Webhook을 수신하고 Payment Provider 조회를 통해 2차 검증하며, 브라우저 단에서 누락된 결제건은 Reconciliation Cron이 찾아내 보정한다. 환불 credit note는 Provider 정산 확인 완료 상태를 검증한 후 자동 발행된다.

**프랑스 법에 맞춘 인보이싱.** 프랑스 법정 Facture/Avoir 번호 규격(CGI art. 242 nonies A)의 누락 및 중복 방지를 위해 Atomic Counter 제어 및 DB Constraint 이중 안전장치를 구축했다. 정산 확정 시 PDF 인보이스 자동 발송, 국세청 현금영수증 내역 export 및 pg_cron 기반 자동 리마인더 파이프라인을 구현했다.

**스튜디오의 하루를 자동화.** 카카오톡, WhatsApp, Instagram, 이메일로 파편화되어 있던 상담 문의를 진행 상태별 자동화 메시지 발송 체계로 통합하고, 사진작가별 Google Calendar 양방향 동기화(Sync)를 구현했다. 가격 설정, 메시지 템플릿, 브랜드 구성을 백오피스 노코드 환경으로 구축하여 스튜디오 운영자가 개발자 의존 없이 시스템을 확장할 수 있게 했다.

---

## 직접 만들고 운영하는 제품

### [Bonjour Admin](https://bonjouradmin.com) | AI Software Engineer & Founder
**2026.02 ~ 현재 · 유료 구독자**

프랑스 거주 외국인은 체류 자격 및 개인 상황이 각기 달라 범용 챗봇의 일반론적인 답변으로 대응하기 어렵고, 오답 발생 시 체류 자격에 직접적인 불이익이 발생한다. Polyfact에서 구축하던 RAG 파이프라인 아키텍처를 발전시켜, 사용자의 개인 체류 상황에 맞춰 프랑스 법령 및 공식 안내만 근거로 정확한 조항과 수치를 확인해 6개 언어로 답변하는 AI 서비스를 구축·운영 및 수익화했다.

`Next.js 16 / React 19` `TypeScript` `Supabase + pgvector` `OpenAI Responses API` `Cohere rerank` `Upstash Redis` `Polar.sh` `PostHog` `Vercel`

**Personalised retrieval.** 질문이 동일하더라도 사용자 프로필(국적, 지역, 체류 자격, 가족 관계, 직업)에 따라 적용 법률이 달라진다. 프로필 데이터를 Tag 및 RRF Boost 조건으로 결합해 Retrieval 범위와 적용 법 체계를 동적으로 선정한다. 사용자가 외국인 체류 포털 데이터를 연동하면, 역공학(Reverse-Engineering)으로 매핑한 110개 이상의 endpoint 표준 규격으로 파싱하여 답변 생성에 반영한다.

**공식 소스만 근거로.** Légifrance 6개 법전 데이터는 일간, 정부 소스 15개는 주간 단위로 수집·갱신한다. 웹 검색은 검증된 55개 Whitelisted 도메인 내부로 제한하며, 법정 수치는 버전 관리 DB를 구축해 Ground Truth 데이터로 활용한다.

**근거가 약할 때를 아는 retrieval.** Retrieval Gate → Semantic Cache → Full-text / Dual-embedding(pgvector) Weighted RRF → Cohere Rerank → Quality Gate 단계별 검증 파이프라인을 거친다. 근거 부족 판단 시 Quality Gate가 1차 검색 결과를 기각하고, Agent가 Tool을 통해 공식 소스를 재탐색한다. Agent Tool-calling은 최대 3라운드로 제한되며, 생성된 답변 초안은 병렬 Verifier 3종(CRAG, Legal Reference Existence, Citation Entailment)이 검증한다. LLM 생성 SQL은 공개 Fact 테이블 4개만 접근 가능한 Anonymous Role로 샌드박싱하여 실행한다.

**비용과 latency를 재고 나서 줄이기.** Stage별 Cost Telemetry 구축 및 분석 결과, Verification 과정이 전체 비용의 80%, Web Pre-search가 전체 Latency의 49%를 점유함을 확인했다. Pre-search 실행 조건을 캐시 만료 시점으로 제어하여 검색 Latency를 18초에서 0.9초로 단축했으며, Follow-up 질의 시 Retrieval Gate를 통해 60~70%의 검색 과정을 우회 처리(Skip)하도록 최적화했다.

**법이 바뀌어도 따라가는 eval.** 100건 이상의 Eval Set과 LLM-as-judge 지표 6종을 수립하고, Deterministic Metrics를 병행하여 검증 정합성을 확보했다. 법령 수치 테스트 케이스는 실행 시점에 버전 관리되는 Facts 테이블에서 정답을 참조하도록 설계하여 법령 개정 시에도 테스트 코드를 수정할 필요가 없는 구조를 만들었다. 매주 CI에서 자동 실행되며 회귀 발생 시 빌드가 실패하도록 처리했다.

**코드 밖의 일까지 혼자.** 파이프라인 구축 전 Fake-door Test를 통해 시장 수요를 검증했다. 기능별 Quota를 제어하는 Polar 구독 인프라, 1,000개 이상의 Programmatic SEO 페이지 구축, PostHog 기반 전환 퍼널 분석, GDPR 및 EU AI Act에 대응하는 3개 언어 약관 작성, INPI 공식 상표 등록까지 수행했다. AI 챗 기능 외에도 행정 마감일 리마인더, 494개 문항 규모의 귀화 시험 트레이너, 서류 분석, 커뮤니티 경험담 기능을 운영 중이다.

### SonnanAI | AI Software Engineer
**2026.01 ~ 현재 · 파리**

파리 현지 교회의 매주 수동 진행되던 한-불 동시통역 프로세스를 자동화하여, 통역사 없이 실시간 자막과 음성을 제공하는 라이브 파이프라인을 구축했다.

`Python` `Flask (SSE)` `Silero VAD` `OpenAI transcribe + local Whisper` `Qwen2.5-7B LoRA` `Ollama` `pytest`

**예배 중에 멈추지 않는 pipeline.** 현장 네트워크의 빈번한 끊김 환경 대응을 위해 전 과정에 결함 허용(Fault-tolerant) Fallback 아키텍처를 설계했다. Silero VAD로 발화를 세그먼트화하고, Cloud STT 장애 시 Local Whisper로 자동 이관하며, 번역 레이어는 Weekly Cache → Local LoRA → Cloud 3단 구조로 전환된다. 8초 Latency 제한 준수를 위한 Request Hedging, Cloud STT Circuit Breaker, 완벽 오프라인 처리 경로를 구현했다.

**직접 만든 corpus로 Qwen2.5-7B LoRA fine-tuning.** 3년 치 한-불 설교 아카이브를 정규화하고 문장 단위로 정렬해 약 8K 쌍의 파라파이프라인 데이터셋을 구축했으며, Qwen2.5-7B LoRA 학습 후 Ollama로 로컬 서빙한다. 라이브 세션마다 도메인 맥락 데이터가 축적되는 Distillation Flywheel을 구축하고, Train/Eval 데이터 누수를 엄격히 격리했다.

**실측으로 정한 guard.** STT 모델의 무음 구간 환각(Hallucination) 발화 생성을 방지하기 위해 실제 예배 음성 데이터에서 실측한 Logprob 분포 기반 필터링을 적용했다(발화 > -0.12, 환각 < -1.3). 사전 구축된 번역 캐시는 23회 예배 데이터 기준 전체 발화의 약 18%를 흡수/절감했다(세션별 3%~52%).

---

## École 42 Paris | IT Architecture Expert, RNCP 7 (석사 수준)
**2023.10 ~ 2026.10 · 파리**

Data and database architecture 전공. 교수나 강의 없이 동료 평가(Peer Review) 기반으로 28개 과제를 수행했다. 외부 라이브러리 없이 밑바닥(Bottom-up)부터 직접 구현하고 출력 결과를 기준값과 대조하는 방식으로 공부했다.

### AI / ML

**[numpy-only MLP](https://github.com/keonwoo98/Multilayer-Perceptron).** Backpropagation 알고리즘을 수식부터 유도해 구현하고 Adam Optimizer 및 L2 Regularization을 적용했다. 수치 미분 기반 Gradient Check를 통해 오차 1.72e-10 수준으로 직접 구현한 경사도를 검증했다. 96% 정확도 대비 Test BCE가 0.33으로 높게 측정되는 현상을 분석한 결과, 난이도 높은 샘플에 대한 모델의 과신(Overconfidence)이 원인임을 밝혀내고 Mini-batch, L2, Early Stopping을 적용해 Test BCE를 0.0499(정확도 98.59%)로 개선했다.

**[EEG brain-computer interface](https://github.com/keonwoo98/Total-perspective-vortex).** 뇌파 신호 기반 운동 의도 분류 과제. Common Spatial Pattern(CSP)을 두 클래스 공분산의 Generalized Eigenvalue Problem으로 정의하고 Jacobi Rotation 고유값 분해를 직접 구현하여 SciPy 결과와 1e-15 수준으로 일치시켰다. CSP 학습을 Cross-Validation Fold 외부에서 수행 시 Test 정보 유출로 성능이 1.0으로 부풀어 오르는 누수(Data Leakage)를 발견하여 Fold 내부 재학습 구조로 수정하고 정직한 0.844 성능을 도출했다.

**[CNN 잎 병해 분류기](https://github.com/keonwoo98/Leaffliction) (PyTorch).** Conv Block 4개 기반의 Custom CNN을 설계했다. 첫 학습의 Validation 정확도 100% 원인을 조사하여 데이터 증강 후 분할로 인해 동일 이미지 변형이 Train/Val 양쪽에 유출된 누수 현상을 규명했다. 분할 후 학습 중에만 증강이 적용되도록 수정하여 Held-out 99.79% 정확도를 확보했으며, 이는 Fine-tuned EfficientNet-B0(99.86%)와 동등한 수준이다.

**[Q-learning snake](https://github.com/keonwoo98/Learn2Slither).** 뱀 머리 기준 4방향 시야 제약 조건하에서 맵 크기에 독립적인 상태 인코딩을 설계하여, 학습에 사용되지 않은 미지의 보드 크기에서도 동일하게 동작하는 에이전트를 구현했다. 그 외 Linear/Logistic Regression 알고리즘을 외부 라이브러리 없이 직접 작성했다.

### Systems, Security, Blockchain

**[x86 kernel 직접 구현](https://github.com/keonwoo98/KFS-3) (C, NASM).** libc 없이 Recursive Page Directory 기반 Higher-half Paging, Multiboot Memory Map 기반 Frame Allocator, kmalloc/vmalloc Heap을 구현했다. Paging 활성화 시점의 저메모리 실행 호환성을 위해 Bootstrap Page Directory에 Identity 매핑과 Higher-half 매핑을 동시 적용했다. QEMU Monitor VGA 메모리 덤프 기반의 검증 체계를 수립하여 표준 출력 장치 부재 환경에서의 디버깅을 수행했다.

**[Rust 오목 엔진](https://github.com/keonwoo98/Gomoku).** Negamax 알고리즘에 Null-move Pruning, Transposition Table, Lazy SMP를 적용하여 한 수당 500ms 이내에 Depth 10~17을 탐색하는 엔진을 개발했다. 탐색 깊이가 Depth 4로 저하되는 국면을 발견하고 회귀 테스트 케이스로 고정하여 원인 분석 및 최적화를 완료했다.

**Solidity 컨트랙트 2개.** OpenZeppelin 없이 BEP-20 토큰 규격을 직접 구현하고, Mint 및 소유권 이전을 2-of-3 Multisig 검증 후 실행하도록 설계했다. 서명자 제거 시 남아있는 인원이 최소 필요 승인 수에 미달하여 컨트랙트가 데드락되는 문제를 방지하기 위해 사전 검증 로직을 구축했다. 두 번째 NFT 프로젝트에서는 SVG 이미지 및 메타데이터를 온체인에서 전량 생성하도록 구현했으며, 문자열 결합 시 발생하는 Stack Too Deep 에러를 방지하고자 생성 함수 분할 및 Yul 파이프라인 컴파일을 적용했다.

**그 외.** Seed 기반 동기화를 통해 동일 피스 순서를 재현하는 멀티플레이어 테트리스(테스트 커버리지 94.58%), Vagrant 및 K3s 기반 클러스터 구축과 ArgoCD GitOps 동기화 인프라 과제, OWASP Top 10 기반 취약점 모의 침투 및 방어 문서화 프로젝트, Flutter 앱 8종을 수행했다.

## École 42 Seoul
**2021.03 ~ 2023.05 · 서울**

**[42 해커톤 대상](https://github.com/keonwoo98/42_Eduthon) (과학기술정보통신부 장관상).** 3인 팀 리드. C 언어 기반 BMP 이미지 처리 교육용 과제, Reference Implementation, Autograder 패키지를 설계하고 42 Amsterdam, Brussels, Paris 캠퍼스에서 발표를 진행했다.

**[C++98로 만든 미니 nginx](https://github.com/keonwoo98/webserv).** Nginx 문법의 설정 파일을 파싱하여 가상 호스트 및 location을 구성하고 kqueue Non-blocking Event Loop 기반으로 Static File, 업로드, CGI를 비동기 처리하는 HTTP/1.1 서버를 구축했다. 로딩 시점 사전 문법 검증으로 잘못된 설정을 서버 기동 전 차단했으며, Server 단 설정을 Location 단으로 자동 상속하여 상위 모듈과의 의존성을 분리했다. 네트워크 분할로 인해 HTTP 헤더 수신이 중단되더라도 파싱 상태를 보존하고 다음 이벤트에서 이어서 처리하는 상태 머신을 설계했다.

**멀티플레이어 Pong 웹 게임 (NestJS + React).** 42 OAuth, 2FA, Socket.io 실시간 채팅(채널, DM, 차단), 랭킹 및 업적 시스템을 갖춘 실시간 대전 아키텍처를 개발했다. 소켓 이벤트 발생과 UI 상태 변화 간 단방향 데이터 흐름(Unidirectional Data Flow)을 설계해 실시간 동기화 정합성을 유지했다.

**그 외.** Libc 함수 재구현, Bash Subset Shell, Dining Philosophers 동시성 과제, Raycasting 기반 FPS 엔진(C), C++98 STL Container 직접 구현, Hardened Debian VM 및 Docker 기반 인프라 과제를 완성했다.

---

## 기술

- **Open source**: CUBRID (2021): 스토리지 엔진의 Double Write Buffer 병목을 프로파일링해 최적화 패치를 upstream에 기여, 코어 팀에 결과 발표.
- **Proficient**: TypeScript, Python, C/C++, SQL (PostgreSQL)
- **Experienced with**: Rust, Deno, Solidity, Dart/Flutter, x86 assembly
- **AI systems**: RAG (hybrid retrieval, RRF, reranking), agentic tool loops, CRAG, evals (LLM-as-judge), LLM cost and latency engineering, output sandboxing, LoRA fine-tuning, local serving (Ollama), speech pipelines (VAD, STT, TTS)
- **Backend & Web**: Next.js, FastAPI, Flask, SvelteKit, Node/Deno, Supabase/Postgres (RLS, PL/pgSQL), Redis, SSE/WebSocket
- **Infrastructure**: GCP Cloud Run, Vercel, Docker, K3s + ArgoCD, GitHub Actions, Sentry, OpenTelemetry, PostHog
- **언어·병역**: 한국어(모국어) · 영어(업무) · 프랑스어(기초) · 군필(육군 헌병, 2019.01 ~ 2020.08)

