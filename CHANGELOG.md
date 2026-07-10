# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [0.3.3] - 2026-07-10

### Changed
- **OpenCode-only stance**: Removed all references to "Codex Light Edition" / `npx lazycodex-ai install` as a ULW runtime.
  - v0.3.2 introduced a comparison table suggesting Codex could run a "Light Edition" of ULW. The user correctly flagged this as misleading: `npx lazycodex-ai install` is not a real oh-my-openagent distribution, and ULW does not actually run on Codex.
  - `SKILL.md` Overview: now states ULW runs on **OpenCode only** (the only supported runtime). Light Edition note + Codex mentions removed.
  - `SKILL.md` Setup: "Which Edition Should I Use?" comparison table removed. Install is now a single `bunx oh-my-openagent install` line, with a pointer to the Hermes-side `ulw-routing-in-hermes` skill for the "use `ulw` keyword without OpenCode" case.
  - `README.md` 🧠 핵심 개념: ULW란? section rewritten to state OpenCode-only. "OpenCode vs Codex" comparison section removed.

### Notes
- The previous "Codex + Light Edition" framing was speculative. ULW is fundamentally an **OpenCode + oh-my-openagent** runtime feature. Anything else is a different tool.
- For users who want to *route* a `ulw` keyword request inside Hermes (without OpenCode), the companion skill `ulw-routing-in-hermes` (software-development category) covers that decision tree.

---

## [0.3.2] - 2026-07-10

### Changed
- **Clarify Codex compatibility** in both `SKILL.md` and `README.md`
  - Overview now explicitly names both OpenCode (Ultimate Edition, full 11-agent orchestration) and Codex CLI (Light Edition, simplified multi-agent subset)
  - Added "Which Edition Should I Use?" comparison table in `SKILL.md`
  - Added "OpenCode vs Codex" section in `README.md`
  - Documents that `ulw <task>` keyword works on both, but Team Mode / Hephaestus / built-in MCPs are OpenCode-only

### Notes
- Addresses user-reported ambiguity: "Codex도 사용 가능한 것처럼 되어 있는데" — yes, Light Edition supports the `ulw` keyword. Full 11-agent orchestration is OpenCode-only.

---

## [0.3.0] - 2026-07-10

### Added
- **Hermes Agent installation instructions** in both `SKILL.md` and `README.md`
  - Local install: copy to `~/.hermes/skills/autonomous-ai-agents/hermes-ulw/`
  - In-session install: ask Hermes to install via `skill_manage`
  - Note about skill loader caching at session start (new session required)

### Changed
- `README.md` — Restructured around Hermes Agent use case; "Quick start" now emphasizes this is a Hermes skill, not a standalone CLI tool
- `SKILL.md` — Added "Install This Skill into Hermes Agent" section
- CHANGELOG simplified to current state only; no references to internal drafting iterations

### Notes
- This is the first release tagged for end users. v0.1.0 and v0.2.0 were drafts that never had user-facing announcements.

---

## [0.2.0] - 2026-07-10

### Added
- References restructured around the 11-agent model (orchestration / agents / ulw-loop / configuration)
- Templates for ULW keyword usage (10 prompt patterns)

### Notes
- Drafted but never announced to users.

---

## [0.1.0] - 2026-07-10

### Added
- Initial draft (internal)

### Notes
- Drafted but never announced to users.
