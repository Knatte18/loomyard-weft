# Decision record — creel-hook-delivery-fixes

## Goal

A creel mail handed over at a Stop reaches the agent's context once and is shown to the operator once, and the `TestScheduler_HookWake` blocked-chain subtest no longer depends on how fast the delivery worker runs.

## Scope

- In: the Claude engine's rendering of creel's Stop block, `claudeengine.Claude.BoundaryBlock`, and the renderer seam it implements, `shuttleengine.ToolBoundaryContextRenderer`, so the block can differ by event.
- In: the hidden `hook-read` verb in `internal/creelcli/hookread.go`, which passes the event to the renderer.
- In: `TestScheduler_HookWake` in `internal/creelengine/deliver_test.go`, the rows whose one trigger wakes the delivery worker more than once.
- In: the docs that describe the block, `internal/shuttleengine/claudeengine/doc.go` and `internal/creelcli/doc.go`.
- Out: the PostToolUse delivery, which the board entry reports as reaching the agent once; its rendering stays byte for byte.
- Out: the idle waiter, `hook-wait`, and the bell line; the note `creel-ring-carries-mail` is not acted on.
- Out: the merged Stop hook command in `internal/shuttleengine/claudeengine/settings.go` and its turn-end recording.

## Decisions

### D1 — The second copy is Claude Code's handling of one Stop block, not the waiter hook

Claude Code 2.1.312 turns one Stop hook's `decision: block` into two model messages from the same `reason`: a `Stop hook feedback:` message that continues the turn, and a `hook_blocking_error` attachment rendered as `Stop hook blocking error from command: "<command>": <reason>`.
It also shows the `reason` to the operator as `Stop hook error: <reason>`, beside the `systemMessage` line `Stop says: …`.
The idle waiter's exit 2 carries only the bell line on standard error, so it never repeats the mail.
Rationale: read from the Claude Code bundle's hook handling and matched against `docs/research/creel-hook-channels-2026-10-28/report.md` case 2, which shows both operator lines from one block.

### D2 — A Stop block carries the mail in its additional context, and a fixed pointer as its reason

On a Stop, the block's `reason` is a fixed line that names no message and says the mail is in the hook's additional context.
The mail lines go into `hookSpecificOutput.additionalContext` with `hookEventName` `Stop`, which Claude Code hands the model once as `Stop hook additional context: <text>`, and into `systemMessage`, which the operator sees once as `Stop says: <text>`.
`decision: block` stays, so the turn still goes on after the Stop exactly as before, and the Stop count and turn-end recording are unchanged.
Rationale: the block is the only Stop output proven to keep the turn going;
moving the mail out of the reason leaves its two copies carrying the pointer alone.
Bound: the hook's command text still reaches the model, inside the blocking-error copy of the pointer, since Claude Code names the command of every blocking Stop hook; the mail itself is no longer next to it.

### D3 — The renderer seam takes the event

`BoundaryBlock` takes the `shuttleengine.HookEvent` beside the text, and `hook-read` passes the event its input carried.
A tool-boundary block keeps `reason` and `systemMessage` as the text.
Rationale: only the Claude engine implements the seam, and the event is the one fact the rendering now turns on.

### D4 — The HookWake rows wake the worker once per settled trigger

`settle` in `TestScheduler_HookWake` counts one probe wait after its trigger, which is exact only when the trigger wakes alpha's worker once while the worker sleeps.
A trigger posting several messages wakes it again during a probe, the wake channel buffers that wake, and the extra probe runs after `settle` returned, so every later `settle` in the row waits for the previous trigger's probe.
In the blocked-chain row that trailing probe can read the turn end the row sets for its last step, ring the blocked waiter and leave the woken waiter unrung, which fails the row.
The rows that post several messages in one trigger post them before the scheduler starts, and settle the start, whose new worker probes once.
Rationale: the fix waits for the one probe the row depends on, with no real-time wait, and leaves the production worker unchanged.

## Constraints

- `PATTERN-test-speed`: the tests keep their `tier0` tag, injected clock and `t.Parallel`, and add no real-time sleep.
- `PATTERN-shuttle-provider-seam`: Claude's hook output stays under `internal/shuttleengine/claudeengine`; `hook-read` stays engine-neutral and reaches it through the seam.
- `PATTERN-tolerant-records`: no Go-written record changes.

## Auto-mode assumptions

- A Stop hook's `hookSpecificOutput.additionalContext` reaches the model when the same output blocks: Claude Code's Stop hook output schema accepts it, and the Stop handling turns it into a `hook_additional_context` message.
- The two findings share no cause: the duplicate comes from Claude Code's Stop handling, the flake from the test's wake counting.

## Open risks

- A model may trust mail less in additional context than in a block reason; `docs/research/creel-hook-channels-2026-10-28/report.md` saw Haiku decline a PostToolUse additional context.
  The pointer reason tells the model the mail follows, and the tier3 smoke test `TestSmokeCreelHookDelivery` covers the Stop delivery against a real session.
- Claude Code's wording and attachment handling are not a stable contract; a later version could change where a Stop's additional context lands.

## Acceptance criteria

- D1, D2: a Stop block's JSON carries `decision` `block`, the fixed pointer as `reason`, the mail text as `systemMessage` and as the Stop additional context, and the mail text appears in no other field.
- D3: a tool-boundary block's JSON is unchanged, `reason` and `systemMessage` both the text, and `hook-read` renders the block of the event its input carried.
- D4: the blocked-chain row and the Stop-cap row of `TestScheduler_HookWake` settle each trigger on the one probe it causes, and pass with the worker's probes ordered after the row's steps.
- D2: the docs that describe the block, the Claude engine's and the creel CLI's package docs, state the Stop rendering.

## Notes for the plan writer

- The block JSON is pinned in `TestHookContext` in `internal/shuttleengine/claudeengine/settings_test.go`.
