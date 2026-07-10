# Hermes ULW — oh-my-openagent Ultrawork 가이드

> **ULW = Ultrawork. 한 단어로 모든 에이전트를 활성화하고, 끝날 때까지 멈추지 않는 워크플로우.**

---

## 🍔 한 줄 요약

> **"ulw 한 단어 입력하면, AI 팀이 알아서 다 끝내준다"**

음식으로 비유하면:
- 🍱 **단순 주문** = 그냥 프롬프트 — 셰프 한 명이 요리 하나
- 🍽️ **코스 요리** = `@plan` → `/start-work` — 기획 → 실행 분업
- 🍱🍱🍱 **뷔페 동시** = `ulw` — 11명 셰프 브리가드 동시 활성화
- ♾️ **끝까지** = `ulw-loop` — 다 끝내야 퇴근 (Todo Enforcer가 다시 데려옴)

---

## 🧠 핵심 개념 (30초)

### ULW는 Ultra-Large **Window**가 아니에요
**이전 버전(v0.1.0)에서 제가 잘못 가정한 오류**였고, **v0.2.0에서 정정**합니다. 정확한 뜻은 **Ultrawork**입니다.

### 진짜 ULW란?
**oh-my-openagent (omo) 플러그인이 OpenCode에 추가하는 기능.** `ulw` 또는 `ultrawork`라는 단어를 프롬프트에 붙이면:
1. **모든 에이전트 활성화** — 11개 에이전트가 한꺼번에 깨어남
2. **다 끝날 때까지 안 멈춤** — `ulw-loop`가 100% 완료까지 반복
3. **다 같이 일함** — 기획(Prometheus), 실행(Atlas), 검증(Momus), 워커(Sisyphus 등)

OMO의 표현: **"One word. Every agent activates. Doesn't stop until done."**

### ULW vs 그냥 Claude Code

| | 그냥 Claude Code | ULW (oh-my-openagent) |
|---|---|---|
| 한 가지 일 | ✅ 잘함 | ✅ 잘함 |
| 여러 단계 + 검증 | 사람 손 필요 | **에이전트 팀이 자동** ✅ |
| "다 끝내" 보장 | ❌ | ✅ (ulw-loop) |
| 백엔드 | Claude Code CLI | **OpenCode + 11개 에이전트** |

---

## 🚀 5분 안에 시작하기

### 1단계: OpenCode + OMO 설치

```bash
# OpenCode 설치 (https://opencode.ai/)
# Ultimate Edition (OpenCode + 모든 기능)
bunx oh-my-openagent install

# Light Edition (Codex CLI에서 쓰려면)
npx lazycodex-ai install
```

### 2단계: 인증

OpenCode 내에서 `/connect` (slash command) → 본인이 쓰는 provider 선택:

- Anthropic (Claude)
- OpenAI (GPT)
- Gemini
- Z.ai (GLM)
- OpenCode Zen
- 또는 기타 OpenAI 호환 endpoint

### 3단계: 테스트

OpenCode에서 입력:

```
ulw fix the typo in README.md (line 5)
```

→ **모든 에이전트 활성화 → Sisyphus가 끝낼 때까지 일함**

---

## 🎯 사용법

### 🍱 단순한 일 (ulw 없이)

```
fix the typo in README.md
```

→ 그냥 일함. 셰프 한 명. ulw 키워드 안 붙여도 됨.

### 🍽️ 단발성 복잡 작업

```
ulw add JWT authentication to the API
```

→ ULW 모드. **다 끝낼 때까지 자동 진행**. 잠깐, 정의된 Done 기준 충족하면 멈춤.

### 🍽️ 계획부터 (Prometheus 사용)

```
@plan refactor auth to use role-based access control
```

→ Prometheus(기획자)와 인터뷰 시작. `.omo/plans/*.md` 파일 생성됨. Momus(검증자)가 검토 → OKAY 시점에 종료.

```
/start-work
```

→ Atlas(지휘자)가 계획을 읽고 워커들에게 분배.

### 🍱🍱🍱 Team Mode (병렬 다중 에이전트)

```
# 먼저 설정 활성화
# oh-my-openagent.json에 "team_mode": true 추가

ulw audit the user service for security vulnerabilities
```

→ `security-research` 프로필 자동 활성화: **3 hunter + 2 PoC engineer 병렬**.

---

## 👥 11개 에이전트 한눈에 보기

| 에이전트 | 역할 | 읽기/쓰기 |
|---|---|---|
| **Sisyphus** | 메인 워커 (ulw 기본 진입점) | 둘 다 |
| **Hephaestus** | 자율 심층 워커 (AmpCode 스타일) | 둘 다 |
| **Prometheus** | 기획자 (인터뷰 → `.omo/plans/*.md`) | `.md`만 |
| **Momus** | 검증자 (OKAY / REJECT) | 읽기만 |
| **Atlas** | 지휘자 (계획 읽고 위임) | 둘 다 |
| **Metis** | 사전 컨설턴트 (모호함 잡기) | 읽기만 |
| **Oracle** | 아키텍처 자문 | 읽기만 |
| **Sisyphus-Junior** | 작업 실행자 | 둘 다 |
| **Explore** | 코드베이스 패턴 검색 | 읽기만 |
| **Librarian** | 외부 문서/의존성 검색 | 읽기만 |
| **Multimodal-Looker** | 이미지/PDF 분석 | 읽기만 |

---

## 🎯 판단 가이드: 뭘 쓸까?

```
질문: 이건 어떤 일인가?

├─ 단순/즉시 (오타, 한 줄 수정) → 그냥 프롬프트
│
└─ 복잡한가?
   ├─ Yes, 그리고 어떻게 할지 정확히 알아 → @plan (Prometheus) → /start-work
   ├─ Yes, 그리고 알아서 해줬으면 → ulw (Sisyphus)
   └─ Yes, 깊은 추론 필요 → Hephaestus + ulw
```

**빠른 추천 (90% 케이스)**:
- 단발 복잡 → `ulw <task>`
- 계획부터 → `@plan <task>` → `/start-work`
- 다 끝내야 함 → `ulw` (자동)

---

## 🆘 자주 발생하는 문제

### ❓ "ulw"만 치면 모든 게 다 일어나나요?

→ **네.** 모든 에이전트가 활성화되고, 정의된 Done 기준 충족까지 자동 진행. 단, **모호한 작업에는 무한 루프 가능** — 명확한 Done 기준 주세요.

### ❓ 비용이 무서워요

→ `oh-my-openagent.json`에 예산 한도:

```jsonc
{
  "ulw_loop": {
    "max_cost_usd": 2.0
  }
}
```

→ 또는 단순 작업엔 `ulw` 안 붙이기 (셰프 한 명이 더 쌈).

### ❓ 작업 도중 멈추고 싶어요

```
pause    ← 잠시 멈춤
resume   ← 다시 시작
cancel   ← 완전히 중단
```

### ❓ 진행 상황 어디서 봐?

```
.omo/
├── plans/         ← Prometheus의 계획서
├── notepads/      ← 학습 노트
└── ulw-loop/      ← 반복 로그 + 감사 추적
```

### ❓ 같은 작업을 다시 시키면 똑같이 하나요?

→ **비슷하지만 결정적이지 않음**. Wisdom Accumulation이 누적되어 더 똑똑해지지만, 100% 동일 결과 보장 ❌.

---

## 💰 비용 가이드

| 작업 종류 | 비용 |
|---|---|
| 단순 수정 | $0.01~$0.10 (ulw 없이) |
| `ulw` 단발 복잡 작업 | $0.50~$3.00 |
| `@plan` + `/start-work` 큰 기능 | $1.00~$10.00 |
| Team Mode (보안 감사 등) | $3.00~$15.00 |
| 다중 목표 ulw-loop | $2.00~$20.00 (예산 캡 설정 권장) |

> 💡 **팁**: `max_cost_usd` 항상 설정. 그리고 비용 줄이려면 Sisyphus + `ulw`보다 Hephaestus 피하고, haiku 같은 가벼운 모델 활성화.

---

## 📦 더 깊이 들어가는 참고자료

- 📄 [`SKILL.md`](./SKILL.md) — Hermes가 자동 로드하는 스킬 정의
- 📘 [`references/orchestration.md`](./references/orchestration.md) — 3-layer 구조 상세
- 📘 [`references/agents.md`](./references/agents.md) — 11개 에이전트 역할
- 📘 [`references/ulw-loop.md`](./references/ulw-loop.md) — Ralph Loop 메커니즘
- 📘 [`references/configuration.md`](./references/configuration.md) — OMO 설정 가이드
- 📋 [`templates/ulw-prompt-template.md`](./templates/ulw-prompt-template.md) — 복붙용 프롬프트 10종

---

## 🔐 보안

1. **API 키는 `.env`나 Keychain에만** (절대 깃허브 ❌)
2. **`.omo/` 디렉토리는 로컬** (gitignore 권장)
3. **Team Mode opt-in** — 기본 OFF
4. **`prometheus-md-only` 훅 유지** — Prometheus가 무분별하게 파일 쓰는 것 방지

---

## ⚠️ v0.1.0에서 v0.2.0 정정 노트

이전 버전은 ULW를 "Ultra-Large Window"로 잘못 설명했어요. **v0.2.0에서 정확한 정의로 완전히 다시 작성**했습니다. 자세한 내용은 `CHANGELOG.md` 참고.

---

## 🪪 License

MIT — 자세한 내용 [LICENSE](./LICENSE)
