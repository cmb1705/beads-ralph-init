---
name: Beads-Ralph Workflow
description: This skill should be used when the user asks about "beads workflow", "beads-ralph integration", "context recovery after compaction", "session persistence", "TodoWrite vs beads", "task tracking persistence", or needs guidance on how beads and ralph-loop work together for persistent task management across Claude Code sessions.
version: 1.0.0
---

# Beads-Ralph Workflow Guide

## Overview

Beads-ralph integration provides persistent task tracking that survives context compaction and session boundaries. It combines:

- **Beads** (bd) - Git-based issue tracking with dependency management
- **Ralph-loop** - Iterative task completion with completion promises
- **Claude Code hooks** - Automatic context injection and state preservation

## How the Hooks Work Together

```
┌─────────────────┐
│  Session Start  │──▶ SessionStart hook runs `bd prime`
└────────┬────────┘    (injects beads workflow context)
         │
         ▼
┌─────────────────┐
│  Work Session   │──▶ Use TodoWrite + Beads together
└────────┬────────┘    (bd update, bd comment, bd close)
         │
         ▼
┌─────────────────┐
│  Pre-Compact    │──▶ PreCompact hook runs `bd sync --flush-only`
└────────┬────────┘    (preserves state before compaction)
         │
         ▼
┌─────────────────┐
│   Compaction    │──▶ Context summarized
└────────┬────────┘    (beads state persisted in JSONL)
         │
         ▼
┌─────────────────┐
│  Post-Compact   │──▶ SessionStart hook re-injects `bd prime`
└────────┬────────┘    (context recovered automatically)
         │
         ▼
┌─────────────────┐
│  Session Stop   │──▶ Stop hook warns to sync beads
└─────────────────┘    (bd close, bd sync --flush-only)
```

## TodoWrite vs Beads Decision Matrix

| Use Case | Tool | Reason |
|----------|------|--------|
| Single-session steps | TodoWrite | Immediate visibility |
| Multi-session work | Beads | Survives compaction |
| Complex dependencies | Beads | Has `bd dep` support |
| Quick task list | TodoWrite | Less overhead |
| Team coordination | Beads | Git-based sync |
| Exploratory work | TodoWrite | Temporary tracking |

**Best Practice**: Use both together. TodoWrite for immediate step visibility, beads for persistent tracking.

## Ralph Loop Compatibility

When running in a ralph-loop:

1. **State persists in git/files** - beads survives iterations
2. **Context may reset** - always check `bd ready` at iteration start
3. **Capture progress** - use `bd comment <id> "progress"` to track partial work
4. **Check completion promise** - only output promise when genuinely complete

### Ralph Loop Start Pattern

```bash
# At start of each ralph iteration
bd ready                           # What's available
bd list --status=in_progress       # What's already claimed
```

### Ralph Loop Progress Pattern

```bash
# During work
bd update <id> --status=in_progress
# ... do work ...
bd comment <id> "Progress: completed X, Y still pending"
```

## Context Recovery

If context seems missing after compaction:

```bash
bd prime    # Reload workflow context
bd ready    # Check available work
bd status   # Verify database state
```

## Session Close Protocol

Before stopping any session:

1. **Update status**: `bd close <id>` for completed work
2. **Create issues**: `bd create` for discovered work
3. **Sync state**: `bd sync --flush-only`
4. **Verify**: `bd status`

## Essential Commands

| Command | Purpose |
|---------|---------|
| `bd ready` | Show unblocked work |
| `bd prime` | Reload context after compaction |
| `bd sync --flush-only` | Export to JSONL (no git push) |
| `bd status` | Database overview |
| `bd create --title="..." --type=task --priority=2` | New issue |
| `bd close <id>` | Complete work |
| `bd comment <id> "..."` | Add progress notes |
| `bd list --tree` | Show task hierarchy |
| `bd dep add <child> <parent>` | Add dependency |

## Troubleshooting

### Beads Context Not Loading

1. Check hooks are installed: Review `.claude/settings.local.json`
2. Restart Claude Code (hooks load at session start)
3. Run `bd prime` manually

### Tasks Not Persisting

1. Run `bd sync --flush-only` before stopping
2. Check `.beads/` directory exists
3. Verify `bd status` shows correct counts

### Ralph Loop Not Seeing Progress

1. Ensure progress is in beads, not just TodoWrite
2. Use `bd comment` to record partial work
3. Run `bd sync --flush-only` before loop iteration ends
