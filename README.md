# beads-ralph-init

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](https://github.com/cmb1705/beads-ralph-init/releases)

A Claude Code plugin to initialize beads-ralph integration in any project for persistent task tracking with automatic context recovery.

## What It Does

This plugin sets up the complete beads-ralph integration stack:

- **SessionStart hook** - Runs `bd prime` to inject beads context on session start and after compaction
- **PreCompact hook** - Runs `bd sync --flush-only` to preserve state before context compaction
- **Stop hook** - Reminds to sync beads before ending session
- **CLAUDE.md** - Project-level configuration and workflow guidance (appends to existing)
- **AGENTS.md** - Agent instructions for beads integration (appends to existing)

**Non-destructive installation**: Existing hooks, CLAUDE.md, and AGENTS.md content are preserved. The plugin merges hooks and appends documentation sections with identifiable markers.

## Prerequisites

- [Claude Code](https://claude.ai/code) installed
- [beads (bd)](https://github.com/steveyegge/beads) CLI installed and available in PATH

## Installation

### Option 1: Clone and Use Directly

```bash
git clone https://github.com/cmb1705/beads-ralph-init.git
claude --plugin-dir ./beads-ralph-init
```

### Option 2: Add to Your Plugins Directory

```bash
git clone https://github.com/cmb1705/beads-ralph-init.git ~/.claude/plugins/beads-ralph-init
```

Then add to your Claude Code settings or use:
```bash
claude --plugin-dir ~/.claude/plugins/beads-ralph-init
```

## Usage

### Initialize a Project

```bash
cd /path/to/your/project
/beads-ralph-init:init
```

This will:

1. Initialize beads database (`bd init`) if not present
2. Create or **merge** hooks into `.claude/settings.local.json` (preserves existing hooks)
3. Create `.claude/hookify.beads-reminder.local.md`
4. Create or **append** to `CLAUDE.md` and `AGENTS.md` (preserves existing content)

Use `--force` to overwrite existing beads sections instead of skipping them.

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

## Files Created/Modified

| File | Action | Purpose |
| ---- | ------ | ------- |
| `.beads/*` | Created | Beads database |
| `.claude/settings.local.json` | Merged | SessionStart + PreCompact hooks |
| `.claude/hookify.beads-reminder.local.md` | Created | Stop hook reminder |
| `CLAUDE.md` | Appended | Project configuration (with markers) |
| `AGENTS.md` | Appended | Agent instructions (with markers) |

**Markers**: Beads sections in CLAUDE.md and AGENTS.md are wrapped in HTML comment markers (`<!-- BEADS-RALPH-INTEGRATION-START -->` and `<!-- BEADS-AGENT-INSTRUCTIONS-START -->`) for easy identification and removal.

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

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Related Projects

- [beads](https://github.com/steveyegge/beads) - Git-based issue tracking with dependency support
- [Claude Code](https://claude.ai/code) - AI-powered coding assistant
