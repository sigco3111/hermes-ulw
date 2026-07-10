# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-07-10

### Added
- Initial release
- `SKILL.md` with auto-trigger for "ULW", "OmniCoder ULW" keywords
- Three execution modes:
  - **Print mode** — single-shot `claude -p` with macOS `zsh -i` wrapper
  - **tmux Interactive** — multi-turn TUI orchestration
  - **Parallel tmux** — 2-5 concurrent Claude Code instances with worktree isolation
- `references/` directory with 4 deep-dive docs:
  - `print-mode.md`
  - `tmux-interactive.md`
  - `parallel-tmux.md`
  - `zshrc-setup.md`
- `templates/ulw-prompt-template.md` with 8 copy-paste templates
- `README.md` for end users (비개발자 친화)
- LICENSE (MIT)

### Notes
- Designed for Hermes Agent (https://hermes-agent.nousresearch.com/)
- Auto-loads when user mentions "ULW", "OmniCoder ULW", or "Omni-Coder Ultra-Large Window"
- Routes to Claude Code (Anthropic's CLI)
- Replaces Omni-Coder ULW with three flexible modes + auto-selection
