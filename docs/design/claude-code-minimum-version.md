# Claude Code Minimum Version for Gas Town

Research ref: gt-9me

## Summary

**Minimum supported: v2.0.20** (2025-10-16)
**Recommended: v2.1.3+** (2026-01-09)

Gas Town requires Claude Code features that were added incrementally from
v1.0.38 through v2.1.3. The highest watermark dependency is the **Skills
system** (v2.0.20), which Gas Town uses for crew-commit, ghi-list, pr-list,
and pr-sheriff. The skills/commands merge in v2.1.3 makes both systems work
through a unified interface, which matches Gas Town's current `.claude/skills/`
and `.claude/commands/` layout.

## Incident Context

A user experienced nudge failures and refinery startup issues on what was
reported as "v1.4.0". Note: Claude Code versions follow the pattern `X.Y.Z`
where published versions are 0.2.x (pre-GA), 1.0.x (GA), 2.0.x, and 2.1.x.
There is no v1.4.0 in the release history. The reported version may have been
misread or from a fork. Upgrading to v2.1.101 resolved all issues.

## Feature Dependency Chain

Gas Town depends on Claude Code features that were added in this order:

| Feature | Added In | Gas Town Usage |
|---------|----------|----------------|
| Custom slash commands (`.claude/commands/`) | v0.2.31 (2025-03-05) | /done, /review, /handoff, /patrol, /reaper, /backup |
| Hooks system released | v1.0.38 (2025-06-30) | All lifecycle hooks |
| PreCompact hook | v1.0.53 (2025-07-15) | `gt prime --hook` on context compaction |
| UserPromptSubmit hook | v1.0.57 (2025-07-21) | `gt mail check --inject` (nudge queue drain) |
| Subagent/Agent tool | v1.0.60 (2025-07-24) | Agent delegation in crew workers |
| SessionStart hook | v1.0.62 (2025-07-28) | `gt prime --hook && gt mail check --inject` |
| SlashCommand tool | v1.0.123 (2025-09-23) | Claude self-invoking slash commands |
| `--settings` flag | v1.0.x (exact unknown) | Per-role settings isolation |
| Skills system | v2.0.20 (2025-10-16) | /crew-commit, /ghi-list, /pr-list, /pr-sheriff |
| Skills/commands merged | v2.1.3 (2026-01-09) | Unified skill + command interface |
| Deferred tools (ToolSearch) | v2.1.7 (2026-01-14) | MCP tool lazy loading |
| `skipDangerousModePermissionPrompt` | v2.0.x (exact unknown) | Suppresses permission dialog in settings.json |

## What Breaks at Each Version Floor

### Below v1.0.38 — **Non-functional**
No hooks at all. Gas Town cannot inject context at session start, drain nudge
queues, guard tool calls, or record costs. The agent has no Gas Town identity.

### v1.0.38 to v1.0.61 — **Partially functional**
Hooks work but SessionStart hook is missing. Gas Town must fall back to tmux
nudge-based priming, which is unreliable. PreCompact and UserPromptSubmit hooks
may also be missing depending on exact version.

### v1.0.62 to v1.0.122 — **Functional, no skills**
All 5 hook events work. Custom slash commands work. But no Skills support, so
/crew-commit and other skills won't appear. No SlashCommand tool, so Claude
can't self-invoke slash commands.

### v1.0.123 to v2.0.19 — **Functional, no skills**
SlashCommand tool added. Claude can self-invoke /done, /handoff, etc. But still
no Skills system.

### v2.0.20 to v2.1.2 — **Fully functional (minimum supported)**
Skills system available. All Gas Town features work. However, skills and
slash commands are separate systems — the `.claude/skills/` directory works
but the merge behavior may differ from current expectations.

### v2.1.3+ — **Recommended**
Skills and commands merged into a unified system. This matches Gas Town's
current directory layout and frontmatter conventions.

## Undocumented/Fragile Dependencies

These aren't version-gated but can break across any Claude Code update:

| Dependency | What Gas Town Does | Fragility |
|------------|-------------------|-----------|
| Prompt prefix `❯` (U+276F) | Idle detection via tmux capture | UI string, could change |
| Status bar `⏵⏵` (U+23F5) | Busy detection | Undocumented internal |
| NBSP rendering (U+00A0) | Prompt matching normalization | Changed once already (issues/1387) |
| JSONL transcript format | Cost tracking, seance | Undocumented file format |
| `sessions-index.json` | Session discovery | Undocumented file format |
| Config dir path encoding | Project settings location | Undocumented convention |
| PreToolUse JSON stdin format | Guard scripts parse hook input | Could change |
| `.claude.json` oauthAccount field | Account rotation | Undocumented |
| Keychain service naming (SHA-256) | Credential isolation | Undocumented |

## Recommendations

### 1. Update INSTALLING.md

Change Claude Code from "latest" to a minimum version:

```
| **Claude Code** (default) | >= 2.0.20 | `claude --version` | ... |
```

### 2. Add `gt doctor` check

Create `internal/deps/claude.go` (parallel to `dolt.go` and `beads.go`) with
version parsing and comparison. Add `internal/doctor/claude_binary_check.go`
that warns when Claude Code is below the minimum.

Proposed behavior:
- **Not found**: Skip (Claude Code is optional per INSTALLING.md)
- **Below v2.0.20**: Warning — "Claude Code {version} is below minimum (2.0.20), some features will not work"
- **v2.0.20 to v2.1.2**: OK with note — "Consider upgrading to 2.1.3+ for full skills support"
- **v2.1.3+**: OK

### 3. Add startup warning to `gt prime`

When running inside Claude Code (detectable via CLAUDE_CONFIG_DIR or process
name), `gt prime` could check the version and emit a warning line if below
minimum. This gives agents immediate visibility without waiting for `gt doctor`.

### 4. Document in CLAUDE.md at town root

Add a "Prerequisites" section or link to the version requirements doc so that
new sessions see it during priming.

## Not Recommended

- **Hard enforcement** (blocking startup below minimum): Gas Town's design
  principle is graceful degradation. An old Claude Code still works for basic
  tmux orchestration. Blocking would prevent partial functionality.

- **Pinning to exact versions**: Claude Code releases frequently (200+ npm
  versions). Pinning creates unnecessary upgrade friction.

## Version History Sources

- Claude Code CHANGELOG.md: https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
- npm package: @anthropic-ai/claude-code
- GA release (v1.0.0): 2025-05-22
- v2.0.0 release: 2025-09-29
