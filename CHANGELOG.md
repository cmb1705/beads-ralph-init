# Changelog

All notable changes to the beads-ralph-init plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-01-28

### Added
- Initial release
- `/beads-ralph-init:init` command for project initialization
- `beads-ralph-workflow` skill for automatic guidance
- SessionStart hook for `bd prime` context injection
- PreCompact hook for `bd sync --flush-only` state preservation
- Stop hook (hookify rule) for session close reminder
- CLAUDE.md and AGENTS.md templates
- Comprehensive README documentation
