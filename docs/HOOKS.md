# Hooks

Hooks are shell scripts that fire automatically at lifecycle events in Claude Code.
You never invoke them manually. They provide automated quality enforcement.

## Active Hooks

| Hook | Event | Purpose |
|------|-------|---------|
| `post-edit.js` | PostToolUse | Per-file typecheck on every edit |
| `circuit-breaker.js` | PostToolUseFailure | Detect failure loops |
| `quality-gate.js` | Stop | Anti-pattern scan before session ends |
| `intake-scanner.js` | SessionStart | Report pending work items |
| `protect-files.js` | PreToolUse | Block edits to protected files |
| `pre-compact.js` | PreCompact | Save context before compression |
| `restore-compact.js` | SessionStart (compact) | Restore context after compression |
| `worktree-setup.js` | WorktreeCreate | Initialize agent worktrees |

## Lifecycle Events

| Event | When | Can Block? |
|-------|------|-----------|
| SessionStart | New conversation begins | No |
| PreToolUse | Before a tool is called | Yes (exit 2) |
| PostToolUse | After a tool completes | No |
| PostToolUseFailure | After a tool fails | No |
| PreCompact | Before message compression | No |
| Stop | Session ending | No |
| WorktreeCreate | Agent creates a worktree | No |

## Configuration

Hooks are configured in `.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "node \"$CLAUDE_PROJECT_DIR/.claude/hooks/post-edit.js\"",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

## Language-Adaptive Typecheck

The `post-edit.js` hook detects your project's language from `.claude/harness.json`
and runs the appropriate checker:

| Language | Checker | Per-File? |
|----------|---------|-----------|
| TypeScript | `tsc --noEmit` | Yes |
| Python | `mypy` or `pyright` | Yes |
| Go | `go vet` | Package-level |
| Rust | `cargo check` | Project-level |

Configure in `harness.json`:

```json
{
  "typecheck": {
    "command": "npx tsc --noEmit",
    "perFile": true
  }
}
```

## Circuit Breaker

Tracks tool failures. After 3 failures:
- Suggests alternative approaches
- After 5 trips: escalates to "stop and rethink" message

State stored in `.claude/circuit-breaker-state.json` (gitignored).

**Note:** The failure counter increments on every `PostToolUseFailure` event and resets
when the threshold is hit (tripped). There is no success-reset mechanism — the counter
tracks failures since the last trip, not strictly consecutive failures. In practice this
means scattered failures across a long session can eventually trip the breaker, which is
conservative by design.

## Quality Gate

Scans recently modified files on session end. Built-in rules:

| Rule | What It Catches |
|------|----------------|
| `no-confirm-alert` | `confirm()`, `alert()`, `prompt()` in JS/TS |
| `no-transition-all` | `transition-all` in CSS/JSX |
| `no-magic-intervals` | Hardcoded `setInterval` numbers |

Add custom rules in `harness.json`:

```json
{
  "qualityRules": {
    "builtIn": ["no-confirm-alert", "no-transition-all"],
    "custom": [
      {
        "name": "no-console-log",
        "pattern": "console\\.log\\(",
        "filePattern": "\\.(ts|tsx)$",
        "message": "Remove console.log before committing"
      }
    ]
  }
}
```

## Rules

1. **One hook per lifecycle event.** Don't chain multiple scripts on the same event.
2. **Hooks must be fast.** PostToolUse fires on every edit — keep it under 5 seconds.
3. **Hook output goes to Claude.** Use it for actionable feedback, not noise.
4. **PreToolUse can block.** Exit code 2 prevents the tool from executing.
5. **Everything else is advisory.** Other hooks report but don't block.
