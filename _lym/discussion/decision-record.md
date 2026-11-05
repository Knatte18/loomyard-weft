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

### Added after Discussion (parent, 2026-11-05): Codex idle wake runs in the hook-wait waiter

Decision: The idle wake through codex queue runs in the strand's hook-wait waiter process (WakeIdle, IdleWaker, wakeSession), not in the creel daemon.

Rationale: Claude's idle wake already rides the hook-wait waiter, so this matches the Goal's 'the way claudeengine does' and keeps engine-specific commands out of the creel daemon; the effect the Goal wants, an idle Codex session woken by mail, is the same.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - idle-wake-in-waiter-not-daemon (class decision, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
