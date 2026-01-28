---
name: init
description: Initialize beads-ralph integration in the current project for persistent task tracking with automatic context recovery
argument-hint: "[--force]"
---

# Initialize Beads-Ralph Integration

Set up beads task tracking with ralph-loop compatible hooks for persistent context across sessions.

## What This Command Does

1. **Initialize beads** - Creates `.beads/` database if not present
2. **Configure hooks** - MERGES hooks into existing settings (preserves your config)
3. **Update documentation** - APPENDS to existing CLAUDE.md and AGENTS.md (preserves your content)

## IMPORTANT: Non-Destructive Installation

This command is designed to **preserve existing configurations**:

- Existing hooks in `settings.local.json` are preserved and merged
- Existing content in `CLAUDE.md` and `AGENTS.md` is preserved
- Plugin-specific sections are marked with identifiable headers
- Conflicts are reported to the user with recommended solutions

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

### Step 4: Update settings.local.json (MERGE Strategy)

**CRITICAL: Do NOT overwrite existing settings. MERGE the configuration.**

1. **Check if `.claude/settings.local.json` exists**
2. **If it exists**: Read the existing JSON and merge as follows:
   - Add `"Bash(bd:*)"` to `permissions.allow` array (if not already present)
   - Add the `SessionStart` hook entry to `hooks.SessionStart` array (if not already present)
   - Add the `PreCompact` hook entry to `hooks.PreCompact` array (if not already present)
   - Preserve ALL existing permissions and hooks
3. **If it does NOT exist**: Create with the default content below

**Hooks to add (merge into existing arrays):**

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

**Merge Logic:**

- For `permissions.allow`: Append `"Bash(bd:*)"` if not already in the array
- For `hooks.SessionStart`: Check if a hook with command containing `bd prime` already exists. If not, append the new entry.
- For `hooks.PreCompact`: Check if a hook with command containing `bd sync` already exists. If not, append the new entry.
- If arrays don't exist, create them.

**Conflict Notification:**
If any beads-related hooks are already present, inform the user:
> "Found existing beads hooks in settings.local.json. Skipping duplicate entries. Your existing configuration has been preserved."

### Step 5: Create Stop Hook (Hookify Rule)

This file is plugin-specific with a unique name, so it's safe to create/overwrite:

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

### Step 6: Update CLAUDE.md (APPEND Strategy)

**CRITICAL: Do NOT overwrite existing CLAUDE.md. APPEND to it.**

1. **Check if `CLAUDE.md` exists in the project root**
2. **If it exists**:
   - Search for the marker `<!-- BEADS-RALPH-INTEGRATION-START -->`
   - If marker found: Inform user beads integration already present, skip modification
   - If marker NOT found: APPEND the beads section to the END of the existing file
3. **If it does NOT exist**: Create new file with just the beads content

**Content to append (with markers for identification):**

```markdown

<!-- BEADS-RALPH-INTEGRATION-START -->
## Beads Integration

This project uses **beads** for persistent issue tracking with **ralph-loop** compatibility.

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

### Session Close Protocol

Before stopping:
1. Update beads status for tasks worked on
2. Create issues for discovered work
3. Run `bd sync --flush-only`
4. Verify with `bd status`
<!-- BEADS-RALPH-INTEGRATION-END -->
```

**Conflict Notification:**
If beads integration marker is already found, inform the user:
> "Found existing beads integration section in CLAUDE.md. Skipping to preserve your configuration. To update, remove the section between BEADS-RALPH-INTEGRATION markers and re-run init."

### Step 7: Update AGENTS.md (APPEND Strategy)

**CRITICAL: Do NOT overwrite existing AGENTS.md. APPEND to it.**

1. **Check if `AGENTS.md` exists in the project root**
2. **If it exists**:
   - Search for the marker `<!-- BEADS-AGENT-INSTRUCTIONS-START -->`
   - If marker found: Inform user beads instructions already present, skip modification
   - If marker NOT found: APPEND the beads section to the END of the existing file
3. **If it does NOT exist**: Create new file with just the beads content

**Content to append (with markers for identification):**

```markdown

<!-- BEADS-AGENT-INSTRUCTIONS-START -->
## Beads Issue Tracking

This project uses **bd** (beads) for persistent issue tracking.

### Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View details
bd update <id> --status=in_progress
bd close <id>         # Complete work
bd sync --flush-only  # Sync to JSONL
```

### Session Protocol

#### Starting Work
1. `bd ready` - Check what's available
2. `bd list --status=in_progress` - Check active work
3. `bd update <id> --status=in_progress` - Claim task

#### Ending Session

1. `bd list --status=in_progress` - Review active
2. `bd close <id>` - Close completed
3. `bd sync --flush-only` - Sync state
4. `bd status` - Verify
<!-- BEADS-AGENT-INSTRUCTIONS-END -->
```

**Conflict Notification:**
If beads instruction marker is already found, inform the user:
> "Found existing beads instructions in AGENTS.md. Skipping to preserve your configuration. To update, remove the section between BEADS-AGENT-INSTRUCTIONS markers and re-run init."

### Step 8: Handle --force Flag

If the user provides `--force` argument:
- Overwrite the beads sections in CLAUDE.md and AGENTS.md even if markers are present
- Replace existing beads hooks in settings.local.json
- Inform user that force mode was used

### Step 9: Verify Installation

Run the following to verify:

```bash
bd status
```

### Step 10: Report Summary

Output a detailed summary showing:
- What was created vs what was merged
- Any conflicts detected and how they were handled
- Existing content that was preserved
- Next steps for the user

## Success Message

After completing all steps, output a summary like:

---

**Beads-Ralph Integration Initialized!**

**Actions Taken:**

- `.beads/` - [Created | Already existed]
- `.claude/settings.local.json` - [Created | Merged hooks into existing config]
- `.claude/hookify.beads-reminder.local.md` - [Created | Updated]
- `CLAUDE.md` - [Created | Appended beads section | Skipped (already present)]
- `AGENTS.md` - [Created | Appended beads section | Skipped (already present)]

**Preserved:**

- [List any existing content/hooks that were preserved]

**Conflicts (if any):**

- [List any conflicts detected and how they were resolved]

**Next Steps:**
1. Restart Claude Code to activate hooks
2. Run `bd ready` to see available work
3. Create your first issue: `bd create --title="..." --type=task --priority=2`

---
