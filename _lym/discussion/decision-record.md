# Decision record — creel-mail-delivered-once

## Goal

One creel mail reaches the model once and the operator once, as one message, with no empty turn and no error label, and the hook command text never enters the model's context.
Each hook delivery attempt is logged.

## Scope

In:

- The Stop delivery of `hook-read`: what its output is, so Claude Code shows it once to each side, with no error label and no command text.
- The tool-boundary (PostToolUse) delivery of `hook-read`, which has the same command-text leak and error label.
- The idle ring: `hook-wait` hands the mail over itself, so the woken turn starts with the mail and no empty turn runs.
- The labels Claude Code puts on the ring's wake, for the model and the operator.
- One daemon log line per hook delivery attempt and per bell send.
- The package docs and code comments that state these renderings and bounds, among them `internal/creelengine/doc.go`, `internal/creelcli/doc.go`, `internal/shuttleengine/claudeengine/doc.go` and the ring outcome's comment in `internal/creelengine/pool.go`.

Out:

- The pane path's delivery (notices, reminders, injects) and the bell's text; their attempts are already journaled (`sending` with its outcome, a notice's `read`, `stalled`).
- Any change to the journal's records or to `lym creel log`.

## Decisions

### D1 — A Stop delivery is additional context alone, never a block

On a Stop, `hook-read` prints `hookSpecificOutput.additionalContext` (`hookEventName` `Stop`) holding the claimed lines, with no `decision`, no `reason` and no `systemMessage`, and exits 0.
The merged Stop command keeps its rule: an exit 0 after output records no turn end.

Rationale, from Claude Code 2.1.312's bundled source (the installed binary's `Stop` hook handling):

- A Stop hook's additional context is pushed into the same list as a blocking error, and a non-empty list continues the turn with `stopHookActive` set and the block count raised; so the turn goes on without a block, the next Stop's `stop_hook_active` is true, and Claude Code's own cap of 8 still counts these continuations, so the Stop count and its cap of 6 hold unchanged.
- The model reads it once, as `Stop hook additional context: <lines>`.
- The operator reads it once, as `Stop hook feedback: <lines>` in the stop-hook summary; the summary's `Stop hook error:` lines come only from blocking errors.
- Any block adds the command text and the error label, whatever its reason holds.
- `systemMessage` is what writes the operator's `Stop says:` line; dropping it removes the second operator copy.

### D2 — A tool-boundary delivery is additional context plus the system message, never a block

After a tool call, `hook-read` prints `hookSpecificOutput.additionalContext` (`hookEventName` `PostToolUse`) holding the claimed lines and `systemMessage` holding the same lines, with no `decision` and no `reason`.

Rationale, from the same source: a PostToolUse block also adds the command text and the error label.
Additional context renders to the model once as `PostToolUse:<tool> hook additional context: <lines>` and is not drawn in the terminal, while the system message reaches only the operator, as `PostToolUse:<tool> says: <lines>`.
A tool call's turn goes on whatever the hook prints, so dropping the block changes no turn.

### D3 — The ring hands the mail over

A `hook-wait` request carries a mark that its waiter takes mail; the daemon claims for a ring only to a marked waiter, and rings an unmarked one, an older binary's, as today, with no mail.
When the worker rings a marked waiter, the daemon reads the strand's pause through the seam just before its locked step, as a boundary claim does, and in the locked step claims the strand's mail under the other rules of a boundary claim: hold, `hookFieldCap`, the 8 KiB budget with the first message always handed out, oldest first.
The ring's outcome carries the claimed lines, and they stay reserved until the client acknowledges them on the same connection.
The ring is answered only by that acknowledgement; the claim starts the strand's Stop count over, as a ring does today.
A ring whose claim comes back empty does not ring, and the waiter stays registered.
What the claim leaves past the budget stays unread with its flag standing, so the woken turn's first tool call or its Stop takes it.

`hook-wait` writes the lines, joined by newlines, as the engine's rewake output on stderr, acknowledges, and exits with the wake code only once the acknowledgement is confirmed; `creelengine.BellLine` is no longer its output.
Once a ring carrying mail has arrived it never resumes the wait: a failed write or a failed acknowledgement exits 0, which Claude Code 2.1.312 takes as no wake and whose stderr it drops, so the mail stays unread, the ring stays unanswered, and the bell's 30-second window for an unanswered ring applies as today.
A ring with no lines, from an older daemon during a deploy, also exits 0, and the bell follows.

Bound: a pause that starts between the pause read and the locked step lets one ring deliver, the window a boundary claim already has.
Bound: the mail is journaled read before Claude Code shows it, so a session that dies, or a hook Claude Code ends at its timeout or at session end, between the confirmed acknowledgement and Claude Code reading the exit loses that mail as read, which D5's log line records.
Bound: Claude Code escapes `system-reminder` tags in the wake body and replaces a fixed set of its own marker patterns, so mail holding one of them reaches the model altered, as text in any hook field may.
Claude Code 2.1.312 reads an asyncRewake hook's stderr after the process exits and enqueues it as a notification with priority `next`, which wakes the idle model, so a live session shows what the hook wrote; a ring that races a turn the operator starts lands it behind the operator's prompt, as the Stop path does.

### D4 — The waiter hook carries creel's labels for the wake

The idle waiter's Stop hook entry in `internal/shuttleengine/claudeengine/settings.go` sets `rewakeMessage` to `[creel] Mail that arrived while you were idle:` and `rewakeSummary` to `creel mail`.

Rationale: Claude Code 2.1.312 builds an asyncRewake wake's model text as `<rewakeMessage> <stderr>`, defaulting the prefix to `Stop hook blocking error from command "<hook name>":`, and its operator summary from `rewakeSummary`, defaulting to `Stop hook feedback`.
Both keys are in that version's hook schema, marked `@internal`.
The prefix opens with `[creel]`, so the wake's first line matches the creel trust sentence.

Bound: a session launched before the change keeps its old settings and the default labels until relaunched.

### D5 — The daemon logs each hook delivery attempt and each bell

Every claim through the claim verb or the ring, tool boundary, Stop and ring alike, logs one line through `internal/logger` to the daemon log: the strand, the path (`tool`, `stop`, `ring`), the ids handed out, and the outcome.
The outcome is `acknowledged`, `released` (the client dropped or failed before acknowledging), or nothing handed out with its reason: paused, held by another sender, Stop cap reached, Stop count unknown after a daemon start, or nothing claimable.
Each bell send logs the strand, the claimable ids and the send's outcome.
A claim that hands out mail, every release and every bell log at `Info`; a claim that hands out nothing logs at `Debug`, since a standing flag runs the claim at every tool call.

Bound: a hook that never reaches the daemon, an unreachable socket or a refused claim, is not logged by the daemon; the hook stays silent toward the session as today.

## Constraints

- `PATTERN-pane-text-injection`: no new typing into a pane; the ring delivers through the hook's stderr, and the bell keeps the guarded send.
- `PATTERN-shuttle-provider-seam`: every Claude-specific shape, the JSON keys of D1 and D2, the rewake text and the D4 keys, stays in `internal/shuttleengine/claudeengine`; `hook-read` and `hook-wait` in `internal/creelcli` stay engine-neutral and pass the event or the lines to the seam.
- `PATTERN-spawn-observability` and `internal/logger`'s Level policy govern D5's levels: no `Warn` for a repeated empty claim.
- `PATTERN-tolerant-records`: the waiter's mark of D3 is a request field an older daemon ignores.
- The claim's 8 KiB budget fits every field these outputs use: one hook field holds 10,000 UTF-16 units, the cap creel already measures against.
- The block design of `c49ba448`, as the docs and `TestHookContext` state it, is replaced.
- Each message is still delivered once, by whichever of the tool-boundary claim, the Stop claim, the ring's claim or the pane path reserves it first.
  The one exception is unchanged: `hook-read` writes before it acknowledges, so a failed acknowledgement after its write shows the mail while it stays unread, and it is handed out once more.

## Auto-mode assumptions

- "Is logged" in the title means the daemon log of D5, not a new journal field or a change to `lym creel log`.

## Open risks

- D1 rests on a reading of minified source: the Stop path skips pushing the additional context into the continuation list when a mode check is true, which by its use reads as Claude Code's bare or simple mode, one lym never launches; the tier3 smoke run shows whether a lym session continues the turn.
- D1 and D4 are proven for Claude Code 2.1.312 only: another version may not continue a turn on Stop additional context, which would journal the mail read in a turn that ends, or may refuse the `@internal` keys rather than ignore them; the tier3 smoke run of the acceptance criteria is what proves a new version, and is run again on a Claude Code upgrade.
- A model may weigh mail in additional context less than a block reason; the creel trust sentence names `[creel]` lines, which the mail lines open with, and the smoke run checks the model acts on it.

## Acceptance criteria

- D1: a Stop claim that hands out mail prints JSON with `hookSpecificOutput.additionalContext` holding the lines and no `decision`, `reason` or `systemMessage` key, and `hook-read` exits 0; a Stop claim that hands out nothing prints nothing and exits 1.
- D1: in a real Claude session, a post after the last tool call reaches the model, the turn goes on and acts on it, the events hold one Stop payload for the real end, and the transcript holds no `hook blocking error` text and no hook command text.
- D2: a tool-boundary claim that hands out mail prints JSON with the lines in `additionalContext` and `systemMessage` and no `decision` or `reason` key; in a real session a mid-turn post reaches the model and the transcript holds no hook command text.
- D3: a ring to a marked waiter claims under the boundary rules (budget with the first always handed out, hold, pause, line cap), the outcome carries the lines, and an acknowledgement journals them read and answers the ring; a ring to an unmarked waiter carries no mail and claims nothing.
- D3: a connection dropped before the acknowledgement leaves the mail unread and the ring unanswered, and the bell's unanswered-ring window fires; a ring whose claim would come back empty does not ring and keeps the waiter registered.
- D3: `hook-wait` exits 2 with the lines on stderr only after a confirmed acknowledgement, exits 0 without resuming when the write or the acknowledgement fails, and exits 0 on a ring with no lines.
- D3: in a real Claude session, a post to an idle strand wakes it with the mail in the woken turn's first message, the model acts on it in that turn, and no turn without the mail runs before it.
- D4: the rendered settings carry `rewakeMessage` and `rewakeSummary` on the waiter entry only.
- D5: each claim path logs one line with strand, path, ids and outcome, at `Info` when mail is handed out or released and at `Debug` when nothing is handed out, with each nothing-handed-out reason covered; each bell send logs one `Info` line.

## Notes for the plan writer

- `claudeengine.Claude` renders through the `shuttleengine.ToolBoundaryContextRenderer` seam; the seam's block and rewake methods may need renaming once neither output is a block or a fixed line.
- The Go client today acknowledges an `AckRequired` response by itself when no handler is given; D3's client passes one.
- `TestSmokeCreelHookDelivery` in `internal/creelcli/smoke_llm_test.go` holds the tier3 subtests D1 to D3 extend.
