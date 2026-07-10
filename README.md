# Hermes ULW — OmniCoder ULW 워크플로우 Claude Code로 대체하기

> **Omni-Coder의 ULW(Ultra-Large Window) 기능을 Claude Code CLI로 똑같이 쓰는 법.**

---

## 🍔 한 줄 요약

> **"큰 일은 Claude Code한테 통째로 맡기고, 나는 결과만 확인한다"**

음식으로 비유하면:
- 🍱 **도시락** = `print mode` (한 번에 한 가지 일만 부탁)
- 🍽️ **코스요리** = `tmux 인터랙티브` (여러 단계 거치며 다듬기)
- 🍱🍱🍱 **뷔페** = `병렬 tmux` (여러 가지 일을 한꺼번에 동시에 시키기)

---

## 🧠 핵심 개념 (30초)

### ULW란?
코딩을 시킬 때 **"AI한테 큰 창(컨텍스트)을 한 번에 다 보여주고 그 안에서 일하게 하는"** 방식.
Omni-Coder는 이걸 "초대형 창(ULW)"이라는 이름으로 제공했어요.

### Claude Code란?
**Anthropic의 코딩 전용 도구**. 터미널에 글자를 치면 클로드가 코딩을 함.
내 폴더에 직접 들어가서 파일도 만들고, 테스트도 돌리고, 에러도 고쳐요.

### ULW vs Claude Code

| | ULW (옛날) | Claude Code (이번 워크플로우) |
|---|---|---|
| 작업 위치 | 한 대화창 안에서 다 처리 | 별도 터미널 tmux 세션에서 일함 |
| 동시 작업 | 한 번에 하나씩 | **여러 개 동시 가능** ✅ |
| 비용 측정 | 컨텍스트 토큰 비쌈 | **작업 단위로 측정, 결과만 표시** ✅ |
| 자동 라우팅 | 없음 | **작업 성격에 따라 알아서 모드 선택** ✅ |

> 💡 비유: ULW는 한 손님한테 큰 주방을 다 주는 거고, Claude Code는 **작은 전문 셰프 여러 명**을 부르는 거예요.

---

## 🚀 5분 안에 시작하기

### 1단계: Claude Code 설치 확인

```bash
claude --version
```

✅ 버전이 보이면 OK. ❌ 안 보이면 https://claude.com/claude-code

### 2단계: API 키 세팅

```bash
echo 'export ANTHROPIC_API_KEY="여기에_키"' >> ~/.zshrc && source ~/.zshrc
```

### 3단계: 작동 확인

```bash
zsh -i -c 'claude auth status'
```

`"loggedIn": true` 나오면 준비 끝!

---

## 🍱 방법 1: 도시락 (Print Mode)

**언제?** 한 가지 코딩 작업을 빠르게 시키고 싶을 때 (리뷰, 버그 수정, 문서 생성).

```bash
zsh -i -c 'cd ~/Developer/MyApp && claude -p "로그인 버그 고쳐줘" --max-turns 5'
```

### 자주 쓰는 옵션

| 옵션 | 의미 | 예시 |
|---|---|---|
| `--max-turns N` | 최대 N번 행동 | `--max-turns 5` |
| `--max-budget-usd X` | 비용 상한 | `--max-budget-usd 1` |
| `--model haiku` | 가벼운 모델 | (저렴, 빠름) |
| `--allowedTools "Read"` | 도구 제한 | (안전) |

### 실전 예시

```bash
# 코드 리뷰
git diff main | claude -p "이 변경 리뷰해줘" --max-turns 3

# 버그 수정
zsh -i -c 'cd ~/p && claude -p "로그인 버튼 안 눌리는 거 고쳐줘" --allowedTools "Read,Edit" --max-turns 10'

# README 자동 생성
zsh -i -c 'cd ~/p && claude -p "src/ 구조 보고 README 만들어줘" --allowedTools "Read,Write" --max-turns 8'
```

---

## 🍽️ 방법 2: 코스요리 (tmux 인터랙티브)

**언제?** 큰 기능을 만들면서 단계별로 다듬고 싶을 때.

```bash
# 1. 작업방 만들기
tmux new-session -d -s my-claude -x 140 -y 40

# 2. Claude 띄우기
tmux send-keys -t my-claude 'cd ~/Developer/MyApp && zsh -i -c "claude"' Enter

# 3. 첫 신뢰 다이얼로그 처리 (Enter 누르면 됨)
sleep 5 && tmux send-keys -t my-claude Enter

# 4. 일 시키기
tmux send-keys -t my-claude 'JWT 토큰 쓰도록 auth 모듈 리팩토링해줘' Enter

# 5. 화면 확인
sleep 15 && tmux capture-pane -t my-claude -p -S -50

# 6. 후속 작업 추가
tmux send-keys -t my-claude '이제 유닛테스트도 추가해줘' Enter

# 7. 끝
tmux send-keys -t my-claude '/exit' Enter && tmux kill-session -t my-claude
```

**핵심 슬래시 커맨드** (Claude REPL 안에서):
- `/compact` — 컨텍스트 압축
- `/clear` — 대화 초기화
- `/review` — Claude한테 자기 코드 리뷰 요청
- `/context` — 컨텍스트 사용량 시각화

---

## 🍱🍱🍱 방법 3: 뷔페 (병렬 tmux)

**언제?** 3가지 코딩 작업을 **동시에** 시키고 싶을 때.

```bash
# 작업방 3개 만들기
for i in 1 2 3; do tmux new-session -d -s "task$i" -x 140 -y 40; done

# 각 방에 다른 일 동시에 시키기
tmux send-keys -t task1 'cd ~/p && zsh -i -c "claude -p \"로그인 버그 수정\" --allowedTools \"Read,Edit\" --max-turns 10"' Enter
tmux send-keys -t task2 'cd ~/p && zsh -i -c "claude -p \"API 테스트 추가\" --allowedTools \"Read,Write,Bash\" --max-turns 15"' Enter
tmux send-keys -t task3 'cd ~/p && zsh -i -c "claude -p \"README 업데이트\" --allowedTools \"Read,Edit\" --max-turns 5"' Enter

# 모니터링
sleep 30 && for s in task1 task2 task3; do
  echo "=== $s ===" && tmux capture-pane -t $s -p -S -10
done
```

> 💡 **같은 파일을 동시에 수정하면 충돌!** 각 작업에 `claude -w <feature>` 로 worktree 분리 추천.

---

## 🎯 작업 성격에 따른 모드 자동 선택

Hermes ULW는 사용자가 명시적으로 어느 모드를 쓸지 안 정해도, **작업 성격만 봐서 자동 선택**해요:

| 작업 예시 | 자동 선택 모드 |
|---|---|
| "이 파일 버그 찾아줘" | 🍱 print mode |
| "리팩토링해줘" (큰 변경) | 🍽️ tmux 인터랙티브 |
| "이거 3개 동시에 만들어줘" | 🍱🍱🍱 병렬 tmux |

> **의심스러우면 print mode로 시작** — 가장 안전하고 저럼.

---

## 🆘 자주 발생하는 문제

### ❓ "Invalid API key" 에러

→ `zsh -i -c '...'` 로 감쌌는지 확인. macOS에서 새 bash는 zshrc를 자동 안 읽음.

### ❓ 비용이 무서워요

→ 항상 `--max-budget-usd 0.5 --max-turns 5` 같이 두 개는 같이 적기.

### ❓ 같은 파일 동시 수정 충돌

→ `claude -w <작업명>` 으로 worktree 분리.

### ❓ tmux 첫 신뢰 다이얼로그

→ 1번째 tmux 띄울 때만 `Yes, trust` 다이얼로그 → Enter 누르면 됨.

---

## 💰 비용 가이드

| 작업 종류 | 비용 | 모드 |
|---|---|---|
| 간단 리뷰 | $0.05~$0.20 | print + sonnet |
| 버그 수정 | $0.10~$0.50 | print + sonnet |
| 큰 기능 구현 | $0.50~$3.00 | tmux + sonnet |
| 3개 병렬 | $1.00~$5.00 | tmux 3개 |
| 단순 반복 | $0.01~$0.05 | print + haiku |

---

## 📦 더 깊이 들어가는 참고자료

- 📄 [`SKILL.md`](./SKILL.md) — Hermes가 자동으로 읽는 스킬 정의 파일
- 📘 [`references/print-mode.md`](./references/print-mode.md) — Print Mode 전체 옵션 가이드
- 📘 [`references/tmux-interactive.md`](./references/tmux-interactive.md) — tmux 인터랙티브 완전 가이드
- 📘 [`references/parallel-tmux.md`](./references/parallel-tmux.md) — 병렬 tmux 안전 패턴
- 📘 [`references/zshrc-setup.md`](./references/zshrc-setup.md) — API 키, minimax/OpenRouter 설정
- 📋 [`templates/ulw-prompt-template.md`](./templates/ulw-prompt-template.md) — 복붙용 프롬프트 8종

---

## 🔐 보안

1. **API 키는 `.zshrc`나 Keychain에만**. 깃허브에 절대 ❌
2. **`--allowedTools`로 권한 제한** — `--allowedTools "Read"`면 파일 수정 못 함
3. **민감 폴더에서 직접 실행 금지**

---

## 🪪 License

MIT — 자세한 내용 [LICENSE](./LICENSE)
