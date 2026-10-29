# Creel Stop mail reaches the model once; fix HookWake flake

A creel mail handed over at a Stop now reaches the model once and the operator once.
Before, Claude Code showed the mail twice to both: as `Stop hook feedback:` and again inside `Stop hook blocking error from command: "<command>": <reason>`, and as `Stop says:` and `Stop hook error:` for the operator.
The `TestScheduler_HookWake` blocked-chain subtest also no longer depends on how fast the delivery worker runs.

## Cause

Claude Code turns one blocking Stop hook's `reason` into two model messages and one operator error line.
The idle waiter (`hook-wait`) is not involved: its exit 2 carries only the bell line.
The two findings share no cause: the duplicate comes from Claude Code's Stop handling, the flake from the test's wake counting.

## Change

- On a Stop, `claudeengine.Claude.BoundaryBlock` keeps `decision: block`, so the turn still continues and the Stop count and turn-end recording are unchanged.
  It puts the mail in `hookSpecificOutput.additionalContext` (`hookEventName` `Stop`) and in `systemMessage`.
  The `reason` becomes a fixed pointer line that names no message and says the mail is in the additional context.
  The model reads the mail once as `Stop hook additional context:`, the operator once as `Stop says:`.
- A tool-boundary (PostToolUse) block is byte-for-byte unchanged.
- The `shuttleengine.ToolBoundaryContextRenderer` seam's `BoundaryBlock` now takes the `HookEvent`, and `hook-read` in `internal/creelcli` passes the event its input carried.
  `hook-read` stays engine-neutral.
- `TestScheduler_HookWake`: `settle` counts one probe wait after its trigger, which is exact only when the trigger wakes alpha's worker once.
  A trigger posting several messages woke it again during a probe; the buffered wake ran a probe after `settle` returned and could ring the blocked waiter instead of the woken one.
  The rows that post several messages in one trigger now post them before the scheduler starts and settle the start.
  No real-time wait is added, and the production worker is unchanged.
- The `claudeengine` and `creelcli` package docs state the Stop rendering.
  `TestHookContext` pins the block JSON.

## Design decisions

- The hook's command text still reaches the model, inside the blocking-error copy of the pointer, because Claude Code names the command of every blocking Stop hook.
  The mail itself is no longer next to it.
  Removing the command text entirely would need a Stop output other than a block, and the block is the only Stop output proven to keep the turn going.
- The merged Stop hook command in `settings.go`, `hook-wait` and the bell line are untouched, so the note `creel-ring-carries-mail` is not acted on.

## Check by hand

- A model may trust mail less in additional context than in a block reason; an earlier research run saw Haiku decline a PostToolUse additional context.
  Run the tier3 smoke test `TestSmokeCreelHookDelivery` against a real session; it was not run here.
- That Stop additional context reaches the model alongside a block rests on Claude Code 2.1.312's handling, which is not a stable contract.
  Re-check the delivery after a Claude Code upgrade.
- Run the `TestScheduler_HookWake` blocked-chain subtest repeatedly under CPU load, such as at `tier_parallel` 8, to confirm the flake is gone.
