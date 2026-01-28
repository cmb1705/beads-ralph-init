---
name: init
description: Initialize beads-ralph integration in the current project for persistent task tracking with automatic context recovery
argument-hint: "[--force]"
---

# Initialize Beads-Ralph Integration

Set up beads task tracking with ralph-loop compatible hooks for persistent context across sessions.

## What This Command Does

1. **Initialize beads** - Creates `.beads/` database if not present
2. **Configure hooks** - Sets up SessionStart, PreCompact, and Stop hooks
3. **Create documentation** - Generates CLAUDE.md and AGENTS.md with workflow guidance

## Implementation Steps

### Step 1: Check Prerequisites

First, verify beads is installed:

```bash
bd --version
```

If beads is not installed, inform the user they need to install it first.

### Step 2: Initialize Beads Database

Check if `.beads/` exists. If not, run:

```bash
bd init
```

### Step 3: Create .claude Directory

Ensure `.claude/` directory exists:

```bash
mkdir -p .claude
```

### Step 4: Create settings.local.json

Create `.claude/settings.local.json` with the following content:

```json
{
  "permissions": {
    "allow": [
      "Bash(bd:*)"
    ]
  },
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|compact",
        "hooks": [
          {
            "type": "command",
            "command": "bd prime 2>/dev/null || echo '# No beads database found - run bd init to set up issue tracking'",
            "timeout": 10
          }
        ]
      }
    ],
    "PreCompact": [
      {
        "matcher": "auto|manual",
        "hooks": [
          {
            "type": "command",
            "command": "bd sync --flush-only 2>/dev/null || true",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### Step 5: Create Stop Hook (Hookify Rule)

Create `.claude/hookify.beads-reminder.local.md`:

```markdown
---
name: beads-sync-reminder
enabled: true
event: stop
action: warn
pattern: .*
---

## Session Close Protocol - Beads Sync Required

Before stopping, complete the beads sync protocol:

### Step 1: Update Task Status

```bash
bd list --status=in_progress
bd close <id1> <id2> ...
```

### Step 2: Create Issues for Discovered Work

```bash
bd create --title="Task title" --type=task --priority=2
```

### Step 3: Sync to Git

```bash
bd sync --flush-only
```

### Step 4: Verify

```bash
bd status
```

**IMPORTANT**: If you used TodoWrite during this session, migrate persistent tasks to beads.
```

### Step 6: Create CLAUDE.md

Create `CLAUDE.md` in the project root:

```markdown
# Claude Code Project Configuration

## Project Overview

This project uses **beads** for persistent issue tracking with **ralph-loop** compatibility.

## Beads Integration

### Context Recovery

After compaction, `bd prime` runs automatically via SessionStart hook. If context seems missing:

```bash
bd prime
```

### Core Workflow

1. **Start**: `bd ready` - Check available work
2. **Claim**: `bd update <id> --status=in_progress`
3. **Complete**: `bd close <id>`
4. **Sync**: `bd sync --flush-only`

### TodoWrite vs Beads

- **TodoWrite**: Immediate, single-session visibility
- **Beads**: Persistent, multi-session tracking

## Session Close Protocol

Before stopping:
1. Update beads status for tasks worked on
2. Create issues for discovered work
3. Run `bd sync --flush-only`
4. Verify with `bd status`
```

### Step 7: Create AGENTS.md

Create `AGENTS.md` in the project root:

```markdown
# Agent Instructions

This project uses **bd** (beads) for persistent issue tracking.

## Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View details
bd update <id> --status=in_progress
bd close <id>         # Complete work
bd sync --flush-only  # Sync to JSONL
```

## Session Protocol

### Starting Work
1. `bd ready` - Check what's available
2. `bd list --status=in_progress` - Check active work
3. `bd update <id> --status=in_progress` - Claim task

### Ending Session
1. `bd list --status=in_progress` - Review active
2. `bd close <id>` - Close completed
3. `bd sync --flush-only` - Sync state
4. `bd status` - Verify
```

### Step 8: Verify Installation

Run the following to verify:

```bash
bd status
```

### Step 9: Report Success

Output a summary showing:
- Files created/updated
- Beads database status
- Next steps for the user

## Success Message

After completing all steps, output:

---

**Beads-Ralph Integration Initialized Successfully!**

Created/Updated:
- `.beads/` - Issue tracking database
- `.claude/settings.local.json` - SessionStart + PreCompact hooks
- `.claude/hookify.beads-reminder.local.md` - Stop hook reminder
- `CLAUDE.md` - Project configuration
- `AGENTS.md` - Agent instructions

**Next Steps:**
1. Restart Claude Code to activate hooks
2. Run `bd ready` to see available work
3. Create your first issue: `bd create --title="..." --type=task --priority=2`

---
