## Goal

```markdown
## Seen (operator and hub orch, 2026-11-03)

Creel's hook delivery works very well for Claude sessions.
`codexengine` has none: its package doc says "no creel hooks", so every Codex strand is a pane-mail strand, and mail from another agent reaches it only as a `run lym creel read` reminder.

## What the spike found

`docs/research/codex-hook-delivery-2026-10-28/report.md` tested the channels by hand on Codex CLI 0.164.0:

- A `Stop` hook answering `decision: block` with the message as `reason` is taken at once and acted on.
- A `PostToolUse` block with the message is taken at the next step, but only when the session's instructions say `[creel]` lines are mail; without that sentence the model reads it as a failed tool call.
- An async `Stop` hook cannot wake an idle session; `codex queue --thread <id> --message …` run from outside does, at once.

## Wanted

`codexengine` carries creel's delivery the way `claudeengine` does:

- The tool-boundary claim through a `PostToolUse` block.
- The turn-end delivery through a `Stop` block.
- The idle wake through `codex queue` from the daemon.
- The `[creel]` sentence in the prompt file, so the model reads a block as mail.
- The launch is then marked `CreelHooks`, and the package doc's "no creel hooks" goes.

Each delivery path is covered by a unit test on the hook output and the daemon's wake call; a live check on real Codex sessions is the separate trial task that follows this one.

Related: project `codex-engine-trial`, part 5 (creel by the pane path) and part 7 (stress).
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
