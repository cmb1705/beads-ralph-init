# beads-ralph-init

A Claude Code plugin to initialize beads-ralph integration in any project for persistent task tracking with automatic context recovery.

## What It Does

This plugin sets up the complete beads-ralph integration stack:

- **SessionStart hook** - Runs `bd prime` to inject beads context on session start and after compaction
- **PreCompact hook** - Runs `bd sync --flush-only` to preserve state before context compaction
- **Stop hook** - Reminds to sync beads before ending session
- **CLAUDE.md** - Project-level configuration and workflow guidance
- **AGENTS.md** - Agent instructions for beads integration

## Installation

### From Local Directory

```bash
# Add to your Claude Code settings
claude /plugins add --path "C:\Users\cmb17\.claude\plugins\local\beads-ralph-init"
```

### From Plugin Directory Flag

```bash
claude --plugin-dir "C:\Users\cmb17\.claude\plugins\local\beads-ralph-init"
```

## Usage

### Initialize a Project

```bash
cd /path/to/your/project
/beads-ralph-init:init
```

This will:
1. Initialize beads database (`bd init`)
2. Create `.claude/settings.local.json` with hooks
3. Create `.claude/hookify.beads-reminder.local.md`
4. Create `CLAUDE.md` and `AGENTS.md`

### After Initialization

1. Restart Claude Code to activate hooks
2. Run `bd ready` to see available work
3. Create your first issue: `bd create --title="..." --type=task --priority=2`

## Components

### Command: /beads-ralph-init:init

The main initialization command. Creates all necessary configuration files.

### Skill: Beads-Ralph Workflow

Auto-activating skill that provides guidance when you mention:
- "beads workflow"
- "context recovery"
- "session persistence"
- "TodoWrite vs beads"

## Files Created

| File | Purpose |
|------|---------|
| `.beads/*` | Beads database |
| `.claude/settings.local.json` | SessionStart + PreCompact hooks |
| `.claude/hookify.beads-reminder.local.md` | Stop hook reminder |
| `CLAUDE.md` | Project configuration |
| `AGENTS.md` | Agent instructions |

## How It Works

### Context Recovery Flow

```
Session Start → bd prime → Context Injected
     ↓
Work Session → Use beads + TodoWrite
     ↓
Pre-Compact → bd sync --flush-only → State Preserved
     ↓
Compaction → Context Summarized
     ↓
Post-Compact → bd prime → Context Recovered
     ↓
Session Stop → Hook Reminder → Sync Before Exit
```

### Hook Configuration

**SessionStart** (triggers on startup, resume, compact):
```json
{
  "type": "command",
  "command": "bd prime 2>/dev/null || echo '# No beads database'",
  "timeout": 10
}
```

**PreCompact** (triggers on auto or manual compact):
```json
{
  "type": "command",
  "command": "bd sync --flush-only 2>/dev/null || true",
  "timeout": 30
}
```

## Requirements

- Claude Code
- beads (bd) CLI installed and available in PATH

## Version

1.0.0

## License

MIT
