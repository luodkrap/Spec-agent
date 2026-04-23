# Spec Agent

SDD(Spec Driven Development) 방식으로 스펙 문서를 자동 작성해주는 Claude Code 서브 에이전트 모음입니다.

아이디어만 설명하면 에이전트가 질문을 통해 요구사항을 구체화하고, **requirements → design → tasks** 순서로 구현에 필요한 스펙 문서를 생성해줍니다. 코드는 작성하지 않고 **문서만** 만듭니다.

---

## 두 가지 에이전트

| 에이전트 | 용도 | 템플릿 초점 |
|----------|------|------------|
| [`sdd-spec`](.claude/agents/sdd-spec.md) | 웹/앱 서비스 전용 | 화면 목록 · 컴포넌트 · API 라우트 |
| [`sdd-general`](.claude/agents/sdd-general.md) | 봇 · 자동화 · 파이프라인 · CLI 등 범용 | 입출력 · 모듈 · 외부 연동 · 실행 환경 |

### 어느 것을 쓸까?

- **화면이 있는 서비스** (웹앱, 관리자 페이지, 모바일 앱) → `sdd-spec`
- **그 외 모든 것** (트레이딩 봇, 크롤러, 데이터 파이프라인, 연구 보조 봇, CLI 도구 등) → `sdd-general`
- **헷갈리면** → `sdd-general` (웹/앱도 대응 가능)

---

## 공통 작업 흐름

```
① 요구사항 파악 (에이전트의 질문)
      ↓
② docs/requirements.md 작성 → 사용자 검토
      ↓
③ docs/design.md 작성 → 사용자 검토
      ↓
④ docs/tasks.md 작성 → 사용자 검토
      ↓
⑤ 메인 Claude에게 구현 인계
```

### 생성되는 문서

| 파일 | 내용 |
|------|------|
| `docs/requirements.md` | 목표, 사용자 역할, 입출력/화면, 제약사항, 성공 기준 |
| `docs/design.md` | 기술 스택, 아키텍처, 모듈 구조, 데이터 모델, 외부 연동/API 라우트 |
| `docs/tasks.md` | 선행 관계와 완료 기준이 명시된 구현 작업 목록 (T001, T002, …) |

---

## 설치

### 프로젝트 단위 (이 프로젝트에서만 쓰기)

이 저장소를 열면 [.claude/agents/](.claude/agents/)의 두 에이전트가 자동 인식됩니다. 별도 설치 불필요.

### 전역 (모든 프로젝트에서 쓰기) — 권장

사용자 홈의 `~/.claude/agents/`에 에이전트 파일을 두면 어느 프로젝트에서든 호출됩니다.

```bash
mkdir -p ~/.claude/agents
cp .claude/agents/sdd-spec.md    ~/.claude/agents/
cp .claude/agents/sdd-general.md ~/.claude/agents/
```

최신 상태를 유지하려면 심볼릭 링크가 편합니다.

```bash
ln -sf "$(pwd)/.claude/agents/sdd-spec.md"    ~/.claude/agents/sdd-spec.md
ln -sf "$(pwd)/.claude/agents/sdd-general.md" ~/.claude/agents/sdd-general.md
```

> **우선순위**: 같은 이름의 에이전트가 프로젝트와 전역에 모두 있으면 **프로젝트 로컬이 우선**합니다.

---

## 사용 방법

### 1. 스펙 작성 요청

Claude Code에 만들고 싶은 것을 설명하면 적절한 에이전트가 자동 호출됩니다.

```text
"할 일 관리 앱을 만들고 싶어. 스펙 문서 작성해줘."          → sdd-spec
"비트코인 트레이딩 봇 스펙 작성해줘."                       → sdd-general
"논문 요약 자동화 스크립트 설계 문서 필요해."               → sdd-general
"RSS 피드 수집해서 슬랙으로 보내는 봇 기획해줘."            → sdd-general
"Next.js 블로그 프로젝트 스펙 짜줘."                        → sdd-spec
```

트리거 키워드: `스펙 문서`, `sdd`, `requirements`, `설계 문서`, `봇 만들기`, `자동화`, `트레이딩 봇`, `연구 봇`, `파이프라인`, `스크립트 설계`, `서비스 기획`

### 2. 질문에 답하기

에이전트가 2~3개 질문을 던집니다. 예시:

**sdd-spec의 경우:**
- 어떤 화면이 필요한가?
- 사용자 역할이 몇 개인가?
- 인증 방식은?

**sdd-general의 경우:**
- 시스템이 무엇을 입력받고 무엇을 출력하는가?
- 실행 환경은? (수동 / cron / 이벤트 트리거)
- 외부 의존성은? (API, LLM, 거래소, DB 등)

### 3. 단계별 검토

각 문서 작성 후 에이전트가 검토를 요청합니다. 수정 사항을 말하거나 "다음 단계로 넘어가" 라고 답하면 됩니다.

### 4. 구현 인계

세 문서가 모두 준비되면 메인 Claude에게 이렇게 요청하세요.

```text
@docs/requirements.md @docs/design.md @docs/tasks.md 를 읽고,
T001부터 순서대로 구현해줘. 각 작업 완료 후 git 커밋해줘.
```

---

## 프로젝트 구조

```
spec_agent/
├── .claude/
│   └── agents/
│       ├── sdd-spec.md       # 웹/앱 서비스용
│       └── sdd-general.md    # 봇 · 자동화 · 파이프라인 · CLI 등 범용
└── README.md
```

---

## 원칙

- 에이전트는 **코드를 작성하지 않습니다** (JS, Python, SQL, Shell 등 불포함)
- 문서는 항상 `docs/` 폴더에 저장됩니다
- 기술 스택이 정해지지 않았다면 일반적인 선택지를 제안하고 사용자 확인을 받습니다
- 기존 프로젝트가 있으면 `CLAUDE.md`, `README.md`, `package.json`, `pyproject.toml` 등을 먼저 읽어 컨텍스트를 파악합니다
- 봇/자동화 프로젝트에서는 **리스크 · 실패 시나리오 · 재시도 정책**을 반드시 문서에 포함합니다
