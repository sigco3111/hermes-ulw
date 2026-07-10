# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [0.2.0] - 2026-07-10

### ⚠️ BREAKING / CORRECTION

**ULW의 의미를 정확하게 정정합니다.**

v0.1.0에서 ULW를 **"Ultra-Large Window"**로 잘못 기술했습니다. **이것은 부정확**합니다.

**정확한 정의**: ULW = **Ultrawork** — oh-my-openagent (formerly oh-my-opencode)가 OpenCode에 추가하는 키워드 기능.
- 단어 하나 (`ulw` 또는 `ultrawork`) 입력
- 11개 에이전트 (Sisyphus, Prometheus, Momus, Atlas, Oracle, etc.) 모두 활성화
- ULW Loop으로 끝날 때까지 자동 반복
- "One word. Every agent activates. Doesn't stop until done." (oh-my-openagent 공식)

**근거**: https://github.com/code-yeongyu/oh-my-openagent 의 공식 docs/orchestration.md 및 docs/manifesto.md.

### Changed — 완전히 다시 쓴 파일

- `SKILL.md` — 전면 재작성 (Ultrawork 정의를 중심으로)
- `README.md` — 비개발자용 가이드 새로 작성, v0.1.0 오류 정정 노트 추가
- `references/orchestration.md` (NEW) — 3-layer 아키텍처 가이드 (v0.1.0의 print-mode.md 대체)
- `references/agents.md` (NEW) — 11개 에이전트 상세 (v0.1.0의 tmux-interactive.md 대체)
- `references/ulw-loop.md` (NEW) — Ralph Loop 메커니즘 (v0.1.0의 parallel-tmux.md 대체)
- `references/configuration.md` (NEW) — `oh-my-openagent.json` 설정 가이드 (v0.1.0의 zshrc-setup.md 대체)
- `templates/ulw-prompt-template.md` — ULW 키워드 사용한 프롬프트 템플릿 10종

### Removed (replaced)

- `references/print-mode.md` (v0.1.0) — Claude Code 단발성 모드 가이드 (제거됨)
- `references/tmux-interactive.md` (v0.1.0) — Claude Code 인터랙티브 가이드 (제 제거)
- `references/parallel-tmux.md` (v0.1.0) — Claude Code 병렬 가이드 (제거됨)
- `references/zshrc-setup.md` (v0.1.0) — Claude Code 환경 설정 (제거됨)

> 위 파일들은 v0.1.0의 잘못된 가정 (ULW = Claude Code 대체 워크플로우) 기반이었으므로 제거.

### Why This Happened (v0.1.0의 잘못된 가정)

v0.1.0 작성 시 사용자가 "OmniCoder ULW" 워크플로우를 Claude Code로 대체한다고 안내. 작성자(에르, AI 어시스턴트)가:
1. ULW가 무엇의 약자인지 검증하지 않고 추측
2. "Ultra-Large Window"로 가정한 뒤 모든 문서/스킬 작성
3. factchk 또는 공식 source 확인 안 함

→ 사용자가 "ULW는 Ultraworker의 약자"라고 정정.
→ code-yeongyu/oh-my-openagent 공식 문서로 정확 정의 확인.
→ v0.2.0에서 전면 재작성.

### Contributors

- hjshin (sigco3111) — repo owner
- Claude (Hermes Agent assistant "에르") — drafted original (incorrectly), then corrected.

---

## [0.1.0] - 2026-07-10

### Added
- Initial release (now superseded)
- ⚠️ **Was based on incorrect interpretation**: ULW interpreted as "Ultra-Large Window"
- SKILL.md for auto-trigger on "ulw" / "OmniCoder ULW" / "Ultra-Large Window"
- Three "modes" incorrectly presented: Print Mode, tmux Interactive, Parallel tmux
- references/: print-mode.md, tmux-interactive.md, parallel-tmux.md, zshrc-setup.md
- All describing Claude Code CLI workflows (wrong subject for ULW)

### Notes
- This release is **superseded by v0.2.0**.
- The Claude Code CLI workflows covered in v0.1.0 are still valid patterns, just not what ULW means.
