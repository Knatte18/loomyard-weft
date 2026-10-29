# Creel Stop mail reaches the model once; HookWake test made deterministic

Two fixes to creel's hook delivery, in two commits.

## A Stop's mail reaches the model once

Claude Code turns one blocking Stop hook's `reason` into two model messages: `Stop hook feedback:\n<reason>`, and a blocking-error attachment rendered as `Stop hook blocking error from command: "<command>": <reason>`.
It also shows the reason to the operator as `Stop hook error: <reason>`, and the `systemMessage` as a second line, `Stop says: …`.
This was read from the Claude Code 2.1.312 bundle's hook handling and matches case 2 of `docs/research/creel-hook-channels-2026-10-28/report.md`.
So both copies of the mail came from the merged Stop command's one block.
The idle waiter, `hook-wait`, writes only the bell line, so it never wrote the second copy.

`shuttleengine.ToolBoundaryContextRenderer.BoundaryBlock` now takes the `HookEvent`, and `hook-read` passes the event its input carried.
On a Stop, the Claude engine's block is:

```json
{"decision":"block","reason":"<fixed pointer>","systemMessage":<mail>,"hookSpecificOutput":{"hookEventName":"Stop","additionalContext":<mail>}}
```

The model reads the mail once, as `Stop hook additional context: <mail>`, and the operator sees it once, as `Stop says: <mail>`.
`decision: block` stays, so the turn goes on as before, and the daemon's Stop count, `hook-read`'s exit codes and the merged command's turn-end recording are unchanged.
The tool-boundary block is unchanged byte for byte.

The two findings share no cause.
The note `creel-ring-carries-mail` is not touched: the ring and the bell line are unchanged.
One fact bears on it: Claude Code renders an `asyncRewake` hook's exit-2 output to the model as `Stop hook blocking error from command "<hook name>": <stderr>`, so mail carried by the ring would arrive with that prefix too.

## HookWake rows wait for the probe they depend on

`settle` in `TestScheduler_HookWake` waits for one probe wait after its trigger.
A trigger that posts several messages wakes alpha's worker again while it probes.
The wake channel buffers that second wake, and its probe runs after `settle` returned, so every later `settle` in the row waited for the previous trigger's probe.
In the blocked-chain row, that late probe could read the turn end the row sets for its last step and ring the blocked waiter.
The woken waiter then saw the same claimable set and was never rung, and the row failed at `wantKind(t, woken, OutcomeRing)` after its 10-second deadline.

The blocked-chain row and the Stop-cap row now post their messages before the scheduler starts and settle the start, whose new worker probes once.
`settle`'s comment states that it counts one wake.
The production worker is unchanged.

## Design decisions

- **The block stays on a Stop.**
  A Stop output with additional context and no block would keep the hook command out of the model's context.
  Nothing shows that such an output keeps the turn going, though, and if the turn ended the mail would already be journaled read.
  The block is the only Stop output proven to continue the turn.
- **The pointer reason is Claude-specific, so it lives in `claudeengine`**, as `stopBlockReason`.
- **The seam takes the event instead of gaining a second method**, because only the Claude engine implements it and the event is the one fact the rendering turns on.

## Deviations from the board entry

- The board entry asks that the hook command text not appear in the agent's context at all.
  Under a blocking Stop hook that cannot be done, because Claude Code names the command of every blocking Stop hook.
  The command now appears only beside the fixed pointer line, never beside the mail.

## Testing

- `TestHookContext` pins the exact JSON of both blocks, and `TestToolBoundaryRenderer` and the `hook-read` rows of `TestCreelCLI` expect each event's block.
- `lym gate test` tier0 over the diff and tier1 for `internal/creelcli` passed.
- The old and new HookWake rows were each run under `-run TestScheduler_HookWake -count=300 -cpu=1,2,8`.
  The old rows failed 10 of 900 runs, all in the blocked-chain row at the woken waiter's ring.
  The new rows failed none.

## For the reviewer

- The Stop change rests on how Claude Code handles hook output internally, which is not a stable contract.
  The tier3 smoke test `TestSmokeCreelHookDelivery` covers Stop delivery against a real session and was not run here, since llm-tier runs are refused in this session.
  Run its row `a post after the last tool call is delivered by the Stop block and the turn goes on` once before relying on the change.
- A model may trust mail in additional context less than mail in a block reason.
  The creel-hook-channels research saw Haiku decline a PostToolUse additional context.
  The pointer reason tells the model the mail is there and to act on it.
