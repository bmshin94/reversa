# Reversa 전수조사 분석 정리 🔍

> 작성: Claude Code 세션 (2026-09-20)
> 대상 저장소: 이 프로젝트(`reversa`)

## 📎 GitHub 주소

| 구분 | 주소 |
|:---|:---|
| **원본 저장소 (upstream)** | https://github.com/sandeco/reversa |
| **이 저장소 (fork/clone)** | https://github.com/bmshin94/reversa |
| 공식 문서 (EN) | https://sandeco.github.io/reversa/ |
| 공식 문서 (PT) | https://sandeco.github.io/reversa/pt/ |
| 공식 문서 (ES) | https://sandeco.github.io/reversa/es/ |
| 논문 (arXiv) | https://arxiv.org/abs/2605.18684 |
| npm 패키지 | `npx reversa install` |
| 이슈 트래커 | https://github.com/sandeco/reversa/issues |

---

## 1. 한 줄 정의

> **레거시(기존) 코드를 AI 코딩 에이전트가 안전하게 작업할 수 있는 "실행 가능한 명세서(executable specification)"로 역변환해 주는 프레임워크**

- 이름: Reversa (포르투갈어 "역방향")
- 제작자: sandeco (브라질)
- 라이선스: MIT
- 현재 버전: 1.3.3
- 논문: *Reversa: A Reverse Documentation Engineering Framework for Converting Legacy Software into Operational Specifications for AI Agents* — Macedo & da Costa, 2026.05

---

## 2. 폴더 전수조사 결과

```
reversa/
├── bin/reversa.js        CLI 진입점 (npx reversa <command>)
├── lib/                  설치기 로직 (JS 15개 파일)
│   ├── commands/         install, update, status, uninstall, add-engine, export-diagrams
│   ├── installer/        detector(엔진탐지), writer(복사), manifest(SHA-256), policy(안전규칙),
│   │                     prompts, validator, orange-prompts
│   └── utils/            banner, json-safe
├── agents/               ★ 핵심. 71개 에이전트 x SKILL.md (2.3MB, 243개 파일)
├── templates/            설치 시 배포되는 원본
│   ├── engines/          CLAUDE.md, AGENTS.md, .cursorrules, GEMINI.md, .clinerules,
│   │                     .roorules, .windsurfrules, CONVENTIONS.md, amazonq, copilot-instructions
│   ├── hooks/            check-legacy-policy.mjs (Claude Code PreToolUse 하드 가드)
│   ├── state.json        진행 상태 스키마
│   ├── config.toml       프로젝트 설정
│   └── reversa-config.json  레거시 편집 권한 (기본 allowLegacyEdits: false)
├── docs/                 mkdocs 문서 (영어/포르투갈어/스페인어)
├── evolucao/             내부 설계·회고 기록 (bugs, meta-harness,
│                         permissao-de-edicao-codigo-legado, reducao-de-tokens)
├── BRAINSTORMING/        기능 브레인스토밍 메모
├── scripts/              verify-invocation.py, test-installer-transport.mjs,
│                         test-code-express-compat.mjs
└── .github/workflows/    docs.yml, verify-invocation.yml
```

### 가장 중요한 발견

**진짜 알맹이는 `lib/`(JS)가 아니라 `agents/`(마크다운)다.**
JS 코드는 "파일 복사기"일 뿐이고, 제품의 본체는 71개의 정교한 프롬프트(SKILL.md)다.
즉 Reversa는 *코드 제품*이 아니라 **프롬프트 제품 + 멀티엔진 배포 CLI**다.

---

## 3. 해결하는 문제

```
신규 프로젝트: 스펙 작성 → AI가 구현 → OK
레거시 프로젝트: 스펙 없음 → AI가 무엇을 깨면 안 되는지 모름 → 사고
```

수년간 누적된 암묵적 비즈니스 규칙, 문서화되지 않은 아키텍처 결정, 아무도 건드리기 싫어하는
핵심 로직 — 이 지식은 코드 안에 갇혀 있다. Reversa는 이를 꺼내어 AI가 읽을 수 있는
**운영 계약서(operational contract)** 로 바꾼다.

---

## 4. Discovery 파이프라인 (메인 플로우 `/reversa`)

| 단계 | 에이전트 | 역할 |
|:---|:---|:---|
| 1. 정찰(Reconnaissance) | **Scout** | 폴더 구조, 언어, 프레임워크, 의존성, 진입점 매핑 |
| 2. 발굴(Excavation) | **Archaeologist** | 모듈별 알고리즘·제어흐름·자료구조 심층 분석 |
| 3. 해석(Interpretation) | **Detective** | 암묵적 비즈니스 규칙, 소급 ADR, 상태머신, 권한 추출 |
| | **Architect** | C4 다이어그램, 전체 ERD, 연동 맵, 기술부채 종합 |
| 4. 생성(Generation) | **Writer** | 코드 추적성을 가진 명세서 작성 |
| 5. 검토(Review) | **Reviewer** | 모순 검출, 갭 검증 |

독립 실행 에이전트: **Visor**(스크린샷 기반 UI 스펙), **Data Master**(DB 심층),
**Design System**(디자인 토큰), **Soul Extractor**(`soul.md` 요약), **Reconstructor**(재구축 계획)

각 단계 사이에 `CONTINUAR` 체크포인트가 있어 사용자가 통제권을 유지한다.

---

## 5. 10개 팀 / 71개 에이전트

| 팀 | 진입 명령어 | 목적 |
|:---|:---|:---|
| Reversa Agents Core (Discovery) | `/reversa` | 레거시 분석 → 스펙 생성 (필수) |
| Ideation Agents | `/reversa-brainstorm` | Framer→Explorer→Challenger→Arbiter→Pre-Spec |
| Code New Project Agents | `/reversa-new` | Ideator→Researcher→Drafter→Spec SDD |
| Code Forward Agents | `/reversa-forward` | requirements→clarify→quality→plan→to-do→audit→coding→sync |
| Migration Agents | `/reversa-migrate` | Paradigm Advisor→Curator→Strategist→Designer→Screen Translator→Inspector |
| Pricing and Size Agents | `/reversa-pricing-*` | 공수/규모/가격 3시나리오 산정 |
| Documentation Team | `/reversa-docs` | Three.js·D3·Highcharts 기반 오프라인 HTML 미니사이트 |
| Bug Agents | `/reversa-debugger` | 인과 추적 가능한 결함 메모리 (SPEC↔CODE↔TEST↔BUG) |
| Code Quality Agents | `/reversa-refactor` | 동작 보존 증명 후 구조 개선 (8종) |
| Translators (opt-in) | `/reversa-n8n` | N8N 워크플로 JSON → SDD 스펙 |

무인 실행: `/reversa-autonomous`, `/reversa-new expresso "<아이디어>"`
(질문을 처음에 몰아서 받고 논스톱 실행. 파괴적·외부 노출 명령은 절대 자동 실행 안 함)

---

## 6. 안전 설계 (3~4중 방어)

`lib/installer/policy.js` 기준, 쓰기 허용 폴더는 다음뿐이다:

```
.reversa/   _reversa_sdd/   _reversa_docs/
_reversa_forward/   _reversa_bugs/   _reversa_refactor/
```

1. **기존 파일 수정·삭제 금지** — 신규 파일만 생성
2. `.reversa/reversa-config.json`의 `allowLegacyEdits`가 **기본 false**
3. Claude Code용 **PreToolUse 훅**(`check-legacy-policy.mjs`)이 Write/Edit를 실행 전 물리 차단
4. 설정 파일 부재·파손·타입 오류 시 **fail-safe**(아무것도 못 씀)
5. **에이전트가 이 설정 파일 자체를 수정할 수 없음** (자물쇠의 자물쇠)
6. `update`는 SHA-256 매니페스트로 사용자가 수정한 파일을 감지해 덮어쓰지 않음
7. `uninstall`은 Reversa가 만든 파일만 삭제

---

## 7. 산출물

```
_reversa_sdd/
├── inventory.md / dependencies.md / code-analysis.md
├── data-dictionary.md / domain.md / state-machines.md / permissions.md
├── architecture.md / c4-context.md / c4-containers.md / c4-components.md
├── erd-complete.md / confidence-report.md / gaps.md / questions.md
├── sdd/[component].md        컴포넌트별 스펙
├── openapi/ user-stories/ adrs/ flowcharts/ sequences/
├── ui/ database/ design-system/ addenda/
└── traceability/
    ├── spec-impact-matrix.md
    └── code-spec-matrix.md
```

포워드 사이클 산출물은 `_reversa_forward/<NNN>-<이름>/`에 분리 저장
(requirements.md, roadmap.md, investigation.md, data-delta.md, actions.md,
progress.jsonl, legacy-impact.md, regression-watch.md, audit/)

### 신뢰도 스케일

| 마크 | 의미 |
|:---:|:---|
| 🟢 CONFIRMED | 코드에서 직접 추출 (파일·라인 인용 가능) |
| 🟡 INFERRED | 패턴에서 추론 (틀릴 수 있음) |
| 🔴 GAP | 코드로 판정 불가 (사람 검증 필요) |

할루시네이션을 없애려 하는 대신 **라벨링**한다는 발상이 핵심.

---

## 8. 지원 엔진 (14종)

| 엔진 | 생성 파일 | 스킬 경로 | 활성화 |
|:---|:---|:---|:---|
| Claude Code ⭐ | `CLAUDE.md` | `.claude/skills/` + `.agents/skills/` | `/reversa` |
| Codex ⭐ | `AGENTS.md` | `.agents/skills/` | `reversa` |
| Cursor ⭐ | `.cursorrules` | `.agents/skills/` | `/reversa` |
| Gemini CLI | `GEMINI.md` | `.agents/skills/` | `/reversa` |
| Windsurf | `.windsurfrules` | `.agents/skills/` | `/reversa` |
| Antigravity | `AGENTS.md` | `.agents/skills/` | `/reversa` |
| Kiro | (없음) | `.kiro/skills/` + `.agents/skills/` | `/reversa` |
| Opencode | `AGENTS.md` | `.agents/skills/` | `reversa` |
| Hermes | `AGENTS.md` | `.agents/skills/` | `reversa` |
| Cline | `.clinerules` | `.agents/skills/` | `/reversa` |
| Roo Code | `.roorules` | `.agents/skills/` | `/reversa` |
| GitHub Copilot | `.github/copilot-instructions.md` | `.agents/skills/` | `/reversa` |
| Aider | `CONVENTIONS.md` | `.agents/skills/` | `reversa` |
| Amazon Q Developer | `.amazonq/rules/reversa.md` | `.agents/skills/` | `/reversa` |

---

## 9. 설치 및 사용법

### 사전 준비
- Node.js **18.20.2 이상**
- 백업 3중 권장: `git commit` → 원격 push → `cp -r project project-backup`

### 설치
```bash
cd /path/to/legacy-project
npx reversa install
```
설치기 동작: 엔진 탐지 → 팀 선택 → 프로젝트/사용자 정보 입력 →
`.claude/skills/` 및 `.agents/skills/`로 스킬 복사 → 엔진 진입 파일 생성 →
`.reversa/` 구조 생성 → SHA-256 매니페스트 생성

### 사용
```
/reversa            # 슬래시 지원 엔진
reversa             # Codex 등 슬래시 미지원 엔진
```
세션이 끊겨도 `.reversa/state.json` 체크포인트로 이어서 재개 가능.

### CLI 명령어
```bash
npx reversa install
npx reversa status
npx reversa update
npx reversa add-engine
npx reversa uninstall
npx reversa export-diagrams --format=svg --output=./diagrams   # mermaid-cli 별도 필요
```

---

## 10. 분류: 플러그인 / 스킬 / MCP?

**정답: Agent Skills (+ 배포용 npm CLI).**

| 분류 | 해당 | 근거 |
|:---|:---:|:---|
| MCP 서버 | ❌ | 서버 프로세스·프로토콜 구현 없음 |
| Claude Code 플러그인 | ❌ | `.claude-plugin/plugin.json`·마켓플레이스 등록 없음 |
| **Agent Skills** | ✅ | `SKILL.md` + YAML frontmatter(name/description/license/metadata) 표준 포맷 |
| CLI 설치 도구 | ✅ | npm 패키지 `reversa` = 스킬 복사기 |

특이점: **스킬이 다른 스킬을 읽어 실행하는 오케스트레이션 구조.**
`agents/reversa/SKILL.md`가 "`reversa-[agente]/SKILL.md`를 전문 읽고 현재 컨텍스트에서 실행하라"
고 지시하여, 파일만으로 멀티 에이전트를 구현했다.

MCP가 아니어서 오히려 장점: 설정 0, 14개 엔진 공통, 사용자가 직접 수정 가능.

---

## 11. API 토큰 필요 여부

**필요 없음.**

README 명시: *"Reversa does not request, store, or transmit API keys from any LLM service."*
`lib/` 전체에 HTTP 요청 코드가 없다. 모든 지능은 이미 설치된 AI 에이전트
(Claude Code / Codex / Cursor 등)에 위임된다.

- Reversa 자체 비용: **0원**
- 실제 비용: 사용자의 Claude 등 구독료·토큰 사용량
- 토큰 소모는 적지 않음 → 개발자가 `evolucao/reducao-de-tokens/`에서 절감 작업 진행 중
  (`disable-model-invocation: true`, `references/` 온디맨드 로딩, Reconstructor의 단건 실행 등)

---

## 12. GitHub에서 유명해진 이유 (분석)

1. **시장 공백 정조준** — AI 코딩 툴은 신규 프로젝트에 강하지만 개발자 대다수는 레거시를 다룬다.
2. **arXiv 논문 보유** — "Reverse Documentation Engineering"이라는 개념을 학술적으로 정의, 신뢰도 확보.
3. **벤더 락인 없음** — 14개 엔진 지원으로 확산 범위가 배가.
4. **안전성을 설계로 증명** — 기본 차단 + 훅 + fail-safe. 기업 도입 제안이 가능한 수준.
5. **다국어 + 커뮤니티** — 영/포/스 문서, 브라질 AI 교육 커뮤니티 기반 초기 확산.
6. **시각적 임팩트** — `/reversa-docs`의 Three.js 3D 코드시티·D3·Highcharts·생성형 인장.
7. **진입장벽 제로** — `npx reversa install` 한 줄. 계정·API키·서버·Docker 불필요.
8. **전체 SDLC 커버** — 분석→구현→버그→리팩터링→마이그레이션→견적→문서까지 확장.

> 주의: 실제 스타 수 등 수치는 이 세션에서 확인하지 않았으며, 위 항목은 저장소 내용 기반 분석이다.

---

## 13. 로컬 에이전트 구축에 주는 교훈

| # | 패턴 | 요지 |
|:---:|:---|:---|
| 1 | **파일 기반 상태 관리** | 에이전트의 기억은 컨텍스트가 아니라 디스크(`state.json`)에. 세션 복구·토큰 절약·디버깅 가능 |
| 2 | **물리적 단계 감지** | 메타데이터가 아니라 산출물 존재 여부로 현재 단계를 판정(`reversa-forward`). 상태 불일치 원천 차단 |
| 3 | **신뢰도 마킹** | 할루시네이션을 제거하지 말고 🟢🟡🔴로 라벨링. 사람이 검증 지점을 알 수 있게 |
| 4 | **계층적 컨텍스트 로딩** | SKILL.md는 얇게, 상세는 `references/`에 두고 필요할 때만 읽기 |
| 5 | **다층 방어** | 프롬프트 지시 → 설정 파일 → 훅 물리 차단 → 설정 자체 보호. 프롬프트 가드레일은 뚫린다는 전제 |
| 6 | **SHA-256 안전 업데이트** | 사용자가 수정한 파일을 감지해 덮어쓰지 않음 |

참고할 만한 파일:
- `agents/reversa/SKILL.md` — 오케스트레이터 설계
- `agents/reversa-forward/SKILL.md` — 물리적 단계 감지
- `agents/reversa-debugger-debate/SKILL.md` — 멀티 에이전트 토론 + 독립 심판

권장 디렉터리 골격:
```
my-agent/
├── SKILL.md          # 얇은 진입점
├── references/       # 온디맨드 로딩
├── state.json        # 파일 기반 상태
└── hooks/guard.mjs   # 하드 가드레일
```

---

## 14. React / PHP 관련

### (A) Reversa로 React·PHP 프로젝트를 분석할 수 있나?
**가능.** Scout이 `package.json`, `composer.json`, `requirements.txt`, `pom.xml`, `go.mod`,
`Gemfile`, `Cargo.toml` 등으로 언어를 탐지한다. 언어 중립적 설계.
- PHP 레거시는 사실상 주력 타깃
- React는 Design System(디자인 토큰)·Visor(스크린샷 기반 UI 스펙)가 잘 맞음

### (B) 우리가 Reversa 같은 것을 React·PHP로 만들 수 있나?
Reversa는 3개 층으로 구성된다.

| 층 | 내용 | 포팅 가능성 |
|:---|:---|:---|
| 두뇌 | 71개 SKILL.md (프롬프트) | ✅ 언어 무관. 그냥 마크다운 |
| 설치기 | Node.js CLI | ✅ PHP CLI(composer)로 재작성 가능 |
| 실행 환경 | Claude Code / Cursor 등 | ⚠️ 여기에 지능이 있음 |

- **PHP 버전**: 파일 복사 + 진입 파일 생성이 전부. Laravel 커뮤니티엔 `composer` 배포가 더 자연스러움.
- **React 버전**: CLI보다 대시보드 UI가 의미 있음.
  - 방법 A(추천): `.reversa/state.json` + `_reversa_sdd/**/*.md`를 **읽기만** 해서 시각화. API 키 불필요.
  - 방법 B: React + 백엔드가 Anthropic API를 직접 호출해 SKILL.md를 프롬프트로 주입. SaaS 가능하나 API 비용 발생.

권장 순서: ① 한국어 SKILL.md 팩 → ② React 대시보드(방법 A) → ③ PHP/Laravel 설치기 → ④ SaaS(방법 B)

---

## 15. 수익화 아이디어

라이선스는 MIT이므로 상업적 이용·수정·재배포가 모두 허용된다(저작권 고지 유지 필요).
이름 혼동을 피하기 위해 별도 브랜딩을 권장한다.

### 티어 1 — 자본 0, 즉시 시작

**1) 레거시 진단 서비스 (최우선 추천)**
- 납품물: 아키텍처 보고서(C4), 비즈니스 규칙 목록(🟢🟡🔴), ERD·데이터사전,
  기술부채 우선순위, 리스크 리포트, 3D HTML 문서 사이트, 개선 로드맵·견적
- 가격대: 소형(~3만 LOC) 150~300만원 / 중형(~15만 LOC) 400~800만원 / 대형(50만 LOC+) 1,000~2,000만원
- 근거: 기존엔 시니어 1개월 작업 → Reversa로 실작업 2~3일, 마진 80%+
- 타깃: 담당 개발자 퇴사 기업, M&A 기술 실사, 재구축 검토 기업, 외주 인수인계, 보안 감사 대응
- 영업: Scout 결과만 무료 샘플로 제공 → 전체 보고서 유료 전환

**2) 교육 콘텐츠**
- 온라인 강의 8~15만원 / 워크북·템플릿 팩 3~5만원 / 기업 출강 1일 200~400만원
- 커리큘럼: AI가 레거시에서 실패하는 이유 → SDD 개념 → 설치·첫 분석 → 신뢰도 스케일 →
  커스텀 SKILL.md 작성 → 안전한 기능 추가 → 실전 PHP 쇼핑몰 → 견적 산정
- 강의 수강생 = 진단 서비스 리드

**3) 외주 작업의 무기화 (판매가 아닌 마진 증대)**
- `/reversa` + `/reversa-pricing-*`로 견적 정확도 상승 → 밑지는 계약 방지
- `/reversa-forward`로 구현 속도 상승, 문서 동반 납품으로 단가 인상 명분
- 스펙 보유 → 유지보수 재계약 유리

### 티어 2 — 개발 1~3개월

**4) Reversa Korea (한국화 포크)** — 임팩트 최대
- 71개 SKILL.md 한국어화, 산출물을 한국 SI 문서 양식으로
  (요구사항정의서/화면설계서/테이블정의서/인터페이스정의서/단위테스트결과서)
- 공공 감리 대응 템플릿, 전자정부 프레임워크(eGovFrame) 탐지, JSP/Struts/iBatis, EUC-KR 대응
- 수익: 오픈소스 코어(무료) + 공공 감리 팩 월 5만원 / 금융 컴플라이언스 팩 월 10만원 /
  엔터프라이즈 지원 연 500만원~

**5) Reversa Studio (React 대시보드)**
- `.reversa/state.json` + 스펙 파일을 읽어 진행률·신뢰도·기술부채·리스크 시각화
- 읽기 전용이라 API 키 불필요, 구현 난이도 낮음
- 수익: 무료 로컬 뷰어 / Pro $15·Team $40 (user·월) / 클라이언트 PDF 리포트 건당 과금

**6) 도메인 전문 에이전트 팩**
- 이커머스 50만원 / 핀테크 200만원 / 의료 200만원 / 공공 150만원 / 게임 100만원
- 제작 원가는 사실상 0(마크다운), 진짜 자산은 도메인 지식

### 티어 3 — 본격 사업 3~12개월

**7) SaaS 플랫폼**
- GitHub 연결 → 자동 분석 → 문서 사이트 발행
- Free / Pro $49 / Team $199 / Enterprise 문의
- 킬러 기능: **스펙 드리프트 알림** — PR 머지 시 스펙과 코드 괴리를 감지해
  "이 PR은 domain.md 규칙 R-042를 위반합니다" 자동 코멘트
- 주의: Anthropic API 비용이 직접 발생 → 사용량 기반 과금 설계 필수

**8) 기업 전용 컨설팅 (최고 마진)**
- Phase 1 진단 2주 1,500만원 / Phase 2 전략 2주 2,000만원 /
  Phase 3 PoC 4주 3,000만원 / Phase 4 실행지원 월 1,000만원
- 차별화: PPT가 아니라 **실행 가능한 스펙 + 동작하는 PoC**

**9) 커스텀 에이전트 제작 서비스**
- 사내 컨벤션·API 규격·보안 정책·디자인시스템 준수 검사 에이전트
- 건당 300~1,000만원 + 유지보수 월 50만원

### 실행 로드맵

| 기간 | 할 일 | 목표 |
|:---|:---|:---|
| 0~1개월 | 본인 프로젝트로 직접 실행, 결과물 스크린샷·블로그 3편, SKILL.md 10개 번역 | 경험·콘텐츠 확보 |
| 1~3개월 | 지인 회사 1곳 무료 진단(사례 확보), 상품 페이지, 콘텐츠 정기 발행 | 첫 유료 고객 1건 |
| 3~6개월 | Reversa Korea 오픈소스 공개, React 대시보드 MVP, 강의 오픈 | 월 500~1,000만원 |
| 6~12개월 | 도메인 팩 2~3종, SaaS 베타, 기업 컨설팅 1건 | 월 2,000만원+ |

### 리스크와 대응

| 리스크 | 대응 |
|:---|:---|
| 원작자가 상용 버전 출시 | 한국화·도메인 특화로 로컬 해자 구축 |
| 모델 제공사가 유사 기능 내장 | 본체는 서비스·도메인 지식, 툴은 수단 |
| AI 출력 품질 불안정 | 🟡🔴 마킹 + 사람 검증을 상품에 명시(책임 범위 고지) |
| "AI가 만든 문서" 불신 | 검증 프로세스 자체를 차별화 포인트로 |
| 토큰 비용 | 견적에 반영, Reconstructor 등으로 절감 |

### 최종 추천 순위
1. 레거시 진단 서비스 — 자본 0, 즉시 가능, 마진 최고
2. 한국어 SKILL.md 팩 — 기술 난이도 낮음, 오픈소스 공개로 신뢰·리드 확보
3. React 대시보드 — 기존 강점 활용, API 비용 없음, 데모 효과 우수

> 핵심: 도구를 파는 것이 아니라 **도구로 만든 결과와 신뢰를 파는 것**이다.

---

## 16. 요약 3줄

1. Reversa는 프로그램이 아니라 **AI 에이전트에게 주는 초정밀 업무 매뉴얼 71장**이다.
2. 낡은 코드를 읽어 **AI가 이해할 수 있는 실행 가능한 명세서**로 바꿔 준다.
3. 원본 코드는 **절대 건드리지 않으며**, 다층 방어로 그것을 설계 수준에서 보장한다.
