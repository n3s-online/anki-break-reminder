# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Anki Break Reminder - add-on that tracks study time and prompts breaks with activity suggestions. Targets Anki 2.1.50+.

## Technical Stack

- Python 3
- PyQt5/PyQt6 (via `aqt.qt` for compatibility)
- No external deps - uses Anki's bundled libraries

## Key Anki Add-on Patterns

```python
from aqt import mw           # Main window
from aqt import gui_hooks    # Event hooks
from aqt.qt import *         # Qt widgets (QTimer, QDialog, etc)
from aqt.utils import showInfo, showWarning
from aqt.sound import av_player  # Sound playback
```

Config access:
```python
config = mw.addonManager.getConfig(__name__)
mw.addonManager.writeConfig(__name__, config)
```

## Required Hooks

- `gui_hooks.profile_did_open` - init timer on profile load
- `gui_hooks.reviewer_did_show_question` - detect active reviewing
- `gui_hooks.reviewer_will_end` - pause timer when leaving reviewer

## File Structure (Target)

```
break_reminder/
├── __init__.py      # Entry point, hook registration
├── config.json      # Default settings
├── config.md        # User docs for config
├── timer.py         # QTimer-based study/break timer
├── dialogs.py       # Break popup, settings dialog
├── activities.py    # Activity pool management
├── sounds.py        # av_player wrapper
└── user_files/      # Preserved on upgrade
    ├── break_start.wav
    └── break_end.wav
```

## Testing

Run Anki from terminal to see print() output. Hold Shift while starting Anki to disable all add-ons.

## Distribution

Create `.ankiaddon` zip (contents only, no parent folder). Delete `__pycache__` before zipping.

## Issue Tracking with bd (beads)

This project uses `bd` (beads) for ALL issue tracking. Do NOT use markdown TODOs, task lists, or other tracking methods.

### Setup

If you see "database not found", run:
```bash
bd init --quiet  # Non-interactive, auto-installs git hooks
```

### CLI Quick Reference

```bash
# Find work
bd ready --json                                    # Unblocked issues
bd stale --days 30 --json                          # Forgotten issues

# Create and manage issues
bd create "Issue title" --description="Detailed context about the issue" -t bug|feature|task -p 0-4 --json
bd create "Found bug" --description="What the bug is and how it was discovered" -p 1 --deps discovered-from:<parent-id> --json
bd update <id> --status in_progress --json
bd close <id> --reason "Done" --json

# Search and filter
bd list --status open --priority 1 --json
bd list --label-any urgent,critical --json
bd show <id> --json

# Sync (CRITICAL at end of session!)
bd sync  # Force immediate export/commit/push
```

### Workflow

1. **Check for ready work**: `bd ready` to see what's unblocked
2. **Claim your task**: `bd update <id> --status in_progress`
3. **Work on it**: Implement, test, document
4. **Discover new work**: If you find bugs or TODOs, create issues:
   ```bash
   bd create "Found bug in auth" --description="Login fails with 500 when password has special chars" -t bug -p 1 --deps discovered-from:<current-id> --json
   ```
5. **Complete**: `bd close <id> --reason "Implemented"`
6. **Sync at end of session**: `bd sync`

### IMPORTANT: Always Include Issue Descriptions

Issues without descriptions lack context for future work. When creating issues, always include a meaningful description with:
- **Why** the issue exists (problem statement or need)
- **What** needs to be done (scope and approach)
- **How** you discovered it (if applicable during work)

**Good examples:**
```bash
bd create "Fix auth bug in login handler" \
  --description="Login fails with 500 error when password contains special characters. Found while testing feature X. Stack trace shows unescaped SQL in auth/login.go:45." \
  -t bug -p 1 --deps discovered-from:bd-abc --json

bd create "Add password reset flow" \
  --description="Users need ability to reset forgotten passwords via email. Should follow OAuth best practices and include rate limiting." \
  -t feature -p 2 --json
```

**Bad examples (missing context):**
```bash
bd create "Fix auth bug" -t bug -p 1 --json  # What bug? Where? Why?
bd create "Add feature" -t feature --json     # What feature? Why needed?
```

### Issue Types

- `bug` - Something broken that needs fixing
- `feature` - New functionality
- `task` - Work item (tests, docs, refactoring)
- `epic` - Large feature composed of multiple issues (supports hierarchical children)
- `chore` - Maintenance work (dependencies, tooling)

### Priorities

- `0` - Critical (security, data loss, broken builds)
- `1` - High (major features, important bugs)
- `2` - Medium (nice-to-have features, minor bugs)
- `3` - Low (polish, optimization)
- `4` - Backlog (future ideas)

### Dependency Types

- `blocks` - Hard dependency (issue X blocks issue Y)
- `related` - Soft relationship (issues are connected)
- `parent-child` - Epic/subtask relationship
- `discovered-from` - Track issues discovered during work

Only `blocks` dependencies affect the ready work queue.

### Planning Work with Dependencies

When breaking down large features into tasks, use dependencies to sequence work - NOT phases or numbered steps.

**⚠️ COGNITIVE TRAP: Temporal Language Inverts Dependencies**

Words like "Phase 1", "Step 1", "first", "before" trigger temporal reasoning that flips dependency direction.

**Solution: Use requirement language, not temporal language**

```bash
# ❌ WRONG - temporal thinking leads to inverted deps
bd create "Phase 1: Create buffer layout" ...
bd create "Phase 2: Add message rendering" ...
bd dep add phase1 phase2  # WRONG! Says phase1 depends on phase2

# ✅ RIGHT - requirement thinking
bd create "Create buffer layout" ...
bd create "Add message rendering" ...
bd dep add msg-rendering buffer-layout  # msg-rendering NEEDS buffer-layout
```

**Verification**: After adding deps, run `bd blocked` - tasks should be blocked by their prerequisites, not their dependents.

### Duplicate Detection & Merging

Proactively detect and merge duplicate issues:

```bash
bd duplicates                 # Find all content duplicates
bd duplicates --auto-merge    # Automatically merge all duplicates
bd duplicates --dry-run       # Preview what would be merged
```

### Managing AI-Generated Planning Documents

AI assistants often create planning and design documents during development (PLAN.md, IMPLEMENTATION.md, etc).

**Best Practice**: Store ALL AI-generated planning/design docs in `history/` directory to keep repo root clean.

### Pro Tips

- Always use `--json` flags for programmatic use
- **Always run `bd sync` at end of session** to flush/commit/push immediately
- Link discoveries with `discovered-from` to maintain context
- Check `bd ready` before asking "what next?"
- Use `bd dep tree` to understand complex dependencies
- Priority 0-1 issues are usually more important than 2-4
- Use `--dry-run` to preview changes before applying

### Important Rules

- ✅ Use bd for ALL task tracking
- ✅ Always use `--json` flag for programmatic use
- ✅ Always include `--description` when creating issues
- ✅ Link discovered work with `discovered-from` dependencies
- ✅ Check `bd ready` before asking "what should I work on?"
- ✅ Run `bd sync` at end of session
- ❌ Do NOT create markdown TODO lists
- ❌ Do NOT use external issue trackers
- ❌ Do NOT duplicate tracking systems
