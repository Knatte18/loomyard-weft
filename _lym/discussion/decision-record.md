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
- The operator's copy of the ring's mail, which Claude Code does not draw from the wake.
- One daemon log line per hook delivery attempt and per bell send.
- The package docs and code comments that state these renderings and bounds, among them `internal/creelengine/doc.go`, whose sentence on resuming a dropped hook-wait changes with D3, `internal/creelcli/doc.go`, `internal/shuttleengine/claudeengine/doc.go` and the ring outcome's comment in `internal/creelengine/pool.go`.

Out:

- The pane path's delivery (notices, reminders, injects) and the bell's text; their attempts are already journaled (`sending` with its outcome, a notice's `read`, `stalled`).
- Any change to the journal's records or to `lym creel log`.

## Decisions

### D1 — A Stop delivery is additional context alone, never a block

On a Stop, `hook-read` prints `hookSpecificOutput.additionalContext` (`hookEventName` `Stop`) holding the claimed lines, with no `decision`, no `reason` and no `systemMessage` beyond D6's echo, and exits 0.
The merged Stop command keeps its rule: an exit 0 after output records no turn end.

Rationale, from Claude Code 2.1.312's bundled source (the installed binary's `Stop` hook handling):

- A Stop hook's additional context is pushed into the same list as a blocking error, and a non-empty list continues the turn with `stopHookActive` set and the block count raised; so the turn goes on without a block, the next Stop's `stop_hook_active` is true, and Claude Code's own cap of 8 still counts these continuations, so the Stop count and its cap of 6 hold unchanged.
- The model reads it once, as `Stop hook additional context: <lines>`.
- The operator reads it once, as `Stop hook feedback: <lines>` in the stop-hook summary; the summary's `Stop hook error:` lines come only from blocking errors.
  The summary row clamps the joined lines of every Stop hook to 5, a prevented-continuation reason and `Stop hook error:` lines ahead of the `Stop hook feedback:` lines, so a claim of more than 5 lines, or one beside another hook's output, shows the operator only its head there.
  That meets the Goal: the operator sees the claim once and never twice, and the model, which acts on it, gets it whole.
- Any block adds the command text and the error label, whatever its reason holds.
- `systemMessage` is what writes the operator's `Stop says:` line; keeping the claimed lines out of it removes the second operator copy.
- The push into the continuation list is skipped only in Claude Code's simple mode, the `CLAUDE_CODE_SIMPLE` environment variable or the `--bare` flag, in which the attachment is also dropped before the API call; the same guard stands in the installed 2.1.310, 2.1.311 and 2.1.312.
  No lym session reaches it: lym never passes `--bare`, reed scrubs every `CLAUDE_CODE_`-prefixed variable from the tmux server's environment at boot (`internal/shuttleengine/claudeengine/scrub.go`, applied in `internal/reedengine/lifecycle.go`), and the pane command lym builds sets `CLAUDE_CODE_` variables of its own but never `CLAUDE_CODE_SIMPLE`.
  Only an operator's own Claude settings `env` key could set it, outside lym.

### D2 — A tool-boundary delivery is additional context plus the system message, never a block

After a tool call, `hook-read` prints `hookSpecificOutput.additionalContext` (`hookEventName` `PostToolUse`) holding the claimed lines and `systemMessage` holding D6's echo, when one is owed, ahead of the same lines, with no `decision` and no `reason`.

Rationale, from the same source: a PostToolUse block also adds the command text and the error label.
Additional context renders to the model once as `PostToolUse:<tool> hook additional context: <lines>` and is not drawn in the terminal, while the system message reaches only the operator, as `PostToolUse:<tool> says: <lines>`.
A tool call's turn goes on whatever the hook prints, so dropping the block changes no turn.

### D3 — The ring hands the mail over

A `hook-wait` request carries a mark that its waiter takes mail; the daemon claims for a ring only to a marked waiter, and rings an unmarked one, an older binary's, as today, with no mail.
When the worker rings a marked waiter, the daemon reads the strand's pause through the seam just before its locked step, as a boundary claim does, and in the locked step claims the strand's mail under the other rules of a boundary claim: hold, `hookFieldCap`, the 8 KiB budget with the first message always handed out, oldest first.
The ring's outcome carries the claimed lines, and they stay reserved until the client acknowledges them on the same connection.
The ring's own claim marks the ring answered at the acknowledgement, not at hand-out; the claim starts the strand's Stop count over, as a ring does today.
A ring whose claim comes back empty does not ring, and the waiter stays registered.
What the claim leaves past the budget stays unread with its flag standing, so the woken turn's first tool call or its Stop takes it.
Every way the ring's answer path ends before the daemon journals the acknowledgement releases the claim, among them the client leaving or the daemon stopping after the ring was answered but before the outcome was written.

`hook-wait` writes the lines, joined by newlines, as the engine's rewake output on stderr, acknowledges, and exits with the wake code once the acknowledgement is written; `creelengine.BellLine` is no longer its output.
Once a ring carrying mail has arrived it never resumes the wait: its call leaves the client's reconnect-and-resume path once the ring's outcome has reached its handler, so a connection lost at the acknowledgement or its confirmation ends the call.
A failed write of the lines or of the acknowledgement exits 0, which Claude Code 2.1.312 takes as no wake and whose stderr it drops; the daemon cannot have journaled the read, so the mail stays unread, the ring stays unanswered, and the bell's 30-second window for an unanswered ring applies as today.
Once the acknowledgement is written, `hook-wait` exits with the wake code whether the daemon confirms it, refuses it on a failed journal write, or the confirmation is lost.
A journal write that fails at an acknowledgement, on the ring or any other claim, releases every id the acknowledgement left unjournaled, where today `handOut` leaves the ids after the failing one reserved.
That trades a loss for a duplicate: an exit 0 there would lose as read, unseen, the mail the daemon did journal, while the wake code shows it now, and mail the daemon did not journal stays unread and reaches the model once more at a later claim, the visible duplicate `hook-read` already accepts.
Such a ring stays unanswered, so the bell may follow it once the woken turn ends, as for any unanswered ring.
A ring with no lines, from an older daemon during a deploy, also exits 0, and the bell follows.

Bound: a pause that starts between the pause read and the locked step lets one ring deliver, the window a boundary claim already has.
Bound: the mail is journaled read before Claude Code shows it, so a session that dies, or a hook Claude Code ends at its timeout or at session end, between the journaled read and Claude Code reading the exit loses it as read, recorded by D5's `acknowledged` line.
Bound: an acknowledgement whose confirmation is refused or lost shows the mail the daemon did not journal twice, once in the wake and once at a later claim, and the bell may ring after the woken turn.
Bound: Claude Code escapes `system-reminder` tags in the wake body and replaces a fixed set of its own marker patterns, so mail holding one of them reaches the model altered, as text in any hook field may.
Claude Code 2.1.312 reads an asyncRewake hook's stderr after the process exits and enqueues it as a notification with priority `next`, which wakes the idle model, so a live session's model receives what the hook wrote; a ring that races a turn the operator starts lands it behind the operator's prompt, as the Stop path does.

### D4 — The waiter hook carries creel's labels for the wake

The idle waiter's Stop hook entry in `internal/shuttleengine/claudeengine/settings.go` sets `rewakeMessage` to `[creel] Mail that arrived while you were idle:` and `rewakeSummary` to `creel mail`.

Rationale: Claude Code 2.1.312 builds an asyncRewake wake's model text as `<rewakeMessage> <stderr>`, defaulting the prefix to `Stop hook blocking error from command "<hook name>":`, and its operator summary from `rewakeSummary`, defaulting to `Stop hook feedback`.
Both keys are in that version's hook schema, marked `@internal`.
The wake is enqueued as a task-notification whose summary tag holds the `rewakeSummary` label and whose body, after the tag, holds `<rewakeMessage> <stderr>`.
The terminal draws that notification as one row and never its body: the row component reads the summary, status, duration and a background command's delivery note from the notification, and shows a body only for a monitor event, whose summary has a fixed shape a hook label does not take; neither the expanded nor the verbose view draws it.
The `creel mail` row therefore carries no mail, and D6 gives the operator its copy.
A summary per wake is not open to lym: Claude Code takes a `rewakeSummary` from the hook's stdout JSON only for a hook of a plugin from Anthropic's own marketplaces, and lym's waiter is a settings hook with no plugin, while the settings label is fixed for the session.
The prefix opens with `[creel]`, so the wake's first line matches the creel trust sentence.

Bound: a session launched before the change keeps its old settings and the default labels until relaunched.

### D5 — The daemon logs each hook delivery attempt and each bell

Every claim through the claim verb or the ring, tool boundary, Stop and ring alike, logs one line through `internal/logger` to the daemon log: the strand, the path (`tool`, `stop`, `ring`), the ids handed out, the ids of D6's echo it hands out, and the outcome.
The outcome is `acknowledged`, `released` (the client dropped or failed before acknowledging), `ack-failed` (the client acknowledged and the daemon failed to journal the read, with the error and the ids D3 released, and the ids of the echo it handed out, which D6 drops; a ring's line lists no echo ids, since a ring hands out none), or nothing handed out with its reason: paused, held by another sender, Stop cap reached, Stop count unknown after a daemon start, or nothing claimable.
`released` includes every end of the ring's answer path before the client's acknowledgement reaches the daemon, which D3 releases, the client leaving or the daemon stopping before the outcome was written among them; a confirmation lost after the journaled read logs `acknowledged`.
Each bell send logs the strand, the claimable ids and the send's outcome.
A claim that hands out mail or an echo, every release and every bell log at `Info`; `ack-failed` logs at `Warn`, replacing the daemon's existing journal-failure `Warn` for a claim; a claim that hands out nothing logs at `Debug`, since a standing flag runs the claim at every tool call.

Bound: a hook that never reaches the daemon, an unreachable socket or a refused claim, is not logged by the daemon; the hook stays silent toward the session as today.

### D6 — The ring's mail reaches the operator as an echo at the woken turn's first claim

When the daemon journals a ring's acknowledged read, it keeps the ring's lines and their ids, in memory, as the strand's echo, and the strand's mail flag stands while an echo is owed.
The strand's next claim, tool boundary or Stop, hands the echo out ahead of any mail; a pause, a hold or the Stop cap keeps back the mail and not the echo, since the echo continues no turn.
The echo and the claimed lines share the 8 KiB budget, the first item, echo or message, always handed out, and what does not fit stays owed.
The echo is dropped at the claim's acknowledgement and stays owed at its release.
An acknowledgement whose journal write fails, D5's `ack-failed`, follows D3's ordering: what the hook already showed is dropped, and what the daemon did not journal reaches the operator when a later claim hands it out.
On a claim that hands out an echo, the echo is dropped at that acknowledgement, since `hook-read` printed it before the client acknowledged.
On a ring, the echo keeps the lines of the ids the acknowledgement passed before the failing write, journaled read or skipped as no longer the holder's unread mail, none when the first write fails, and nothing for the failing id and those after it, which D3 released and which reach the operator when a later claim hands them out.
`hook-read` renders the echo through the seam as `systemMessage` under the line `[creel] Delivered while idle:`, so the operator reads it once as a `says:` line and the model not at all: after a tool call ahead of D2's lines, at a Stop beside D1's additional context or alone.
A Stop claim that hands out an echo and no mail prints the `systemMessage` alone and exits 1, and the merged Stop command records the turn end with that output left as the hook's.

Rationale: the terminal never draws the wake's body (D4), the wake's summary cannot carry the lines, and new typing into the pane is barred, so the woken turn's own hooks are the first operator surface lym reaches.

Bound: the echo shows after the model has the mail, at the woken turn's first tool call or its Stop; a woken turn an API error ends fires `StopFailure` instead of `Stop`, which claims nothing, so the echo waits for a later turn's first claim and is never shown when the session ends first.
Bound: a daemon restart drops an owed echo, and the operator then never sees that ring's mail; the model's copy is unaffected, and D5's `ring` line records the ids.

## Constraints

- `PATTERN-pane-text-injection`: no new typing into a pane; the ring delivers through the hook's stderr, and the bell keeps the guarded send.
- `PATTERN-shuttle-provider-seam`: every Claude-specific shape, the JSON keys of D1, D2 and D6, the rewake text and the D4 keys, stays in `internal/shuttleengine/claudeengine`; `hook-read` and `hook-wait` in `internal/creelcli` stay engine-neutral and pass the event or the lines to the seam.
- `PATTERN-spawn-observability` and `internal/logger`'s Level policy govern D5's levels: no `Warn` for a repeated empty claim.
- `PATTERN-tolerant-records`: the waiter's mark of D3 is a request field an older daemon ignores.
- The claim's 8 KiB budget fits every field these outputs use: one hook field holds 10,000 UTF-16 units, the cap creel already measures against.
- The block design of `c49ba448`, as the docs and `TestHookContext` state it, is replaced.
- Each message is still delivered once, by whichever of the tool-boundary claim, the Stop claim, the ring's claim or the pane path reserves it first.
  Two exceptions: `hook-read`, and `hook-wait` once its acknowledgement is written, show the mail whatever the acknowledgement's outcome, so mail the daemon did not journal, released by D3's rule, is handed out once more;
  and the ring's mail is journaled read before Claude Code shows it, so D3's loss window loses it as read.

## Auto-mode assumptions

- "Is logged" in the title means the daemon log of D5, not a new journal field or a change to `lym creel log`.

## Open risks

- D1 and D4 are proven for Claude Code 2.1.312 only, though the Stop continuation guard and the asyncRewake exit-2 rule read the same in the installed 2.1.310, 2.1.311 and 2.1.312.
  Claude Code updates itself, and nothing in lym reruns the smoke on an update.
  A breaking update would journal mail read in a turn that ends, visible only as D5's `acknowledged` line with no continued turn, or would refuse the `@internal` keys rather than ignore them.
  Rerunning the tier3 smoke run after a Claude Code update is an operator action, not a bound.
- A model may weigh mail in additional context less than a block reason; the creel trust sentence names `[creel]` lines, which the mail lines open with, and the smoke run checks the model acts on it.

## Acceptance criteria

- D1: a Stop claim that hands out mail and no echo prints JSON with `hookSpecificOutput.additionalContext` holding the lines and no `decision`, `reason` or `systemMessage` key, and `hook-read` exits 0; a Stop claim that hands out nothing prints nothing and exits 1.
- D1: in a real Claude session, a post after the last tool call reaches the model, the turn goes on and acts on it, the events hold one Stop payload for the real end, and the transcript holds no `hook blocking error` text and no hook command text.
  A claim of more than 5 lines shows the operator its head within the stop-hook summary row's 5 lines and no second copy.
- D2: a tool-boundary claim that hands out mail prints JSON with the lines in `additionalContext` and `systemMessage` and no `decision` or `reason` key; in a real session a mid-turn post reaches the model and the transcript holds no hook command text.
- D3: a ring to a marked waiter claims under the boundary rules (budget with the first always handed out, hold, pause, line cap), the outcome carries the lines, and an acknowledgement journals them read and answers the ring; a ring to an unmarked waiter carries no mail and claims nothing.
- D3: a connection dropped before the acknowledgement leaves the mail unread and the ring unanswered, and the bell's unanswered-ring window fires; the client leaving or the daemon stopping after the ring was answered but before the outcome was written releases the claim; a ring whose claim would come back empty does not ring and keeps the waiter registered.
- D3: a journal write that fails at an acknowledgement releases every id it left unjournaled, so a later claim hands them out.
- D3: `hook-wait` exits 2 with the lines on stderr once its acknowledgement is written, whether the daemon confirms it, refuses it or the confirmation is lost, and never resends the wait; it exits 0 when the write of the lines or of the acknowledgement fails, and on a ring with no lines.
- D3: in a real Claude session, a post to an idle strand wakes it with the mail in the woken turn's first message, the model acts on it in that turn, and no turn without the mail runs before it, and the operator sees one `creel mail` row with no mail under it and the mail once, as D6's echo.
- D4: the rendered settings carry `rewakeMessage` and `rewakeSummary` on the waiter entry only.
- D5: each claim path logs one line with strand, path, ids, echo ids and outcome, at `Info` when mail or an echo is handed out or released, at `Warn` for `ack-failed`, and at `Debug` when nothing is handed out, with each nothing-handed-out reason covered; each bell send logs one `Info` line.
- D6: an acknowledged ring leaves an echo and a standing flag; the strand's next claim hands the echo out ahead of its mail under the shared budget, also when a pause, a hold or the Stop cap keeps the mail back; the acknowledgement drops it and a release keeps it; an acknowledgement whose journal write fails drops a handed-out echo, and on a ring leaves as the echo the lines of the ids before the failing write, journaled or skipped, and none of the ids D3 released; `hook-read` renders it as `systemMessage` alone after the `[creel] Delivered while idle:` line, and a Stop with an echo and no mail exits 1 with that output while the merged Stop command records the turn end.

## Notes for the plan writer

- `claudeengine.Claude` renders through the `shuttleengine.ToolBoundaryContextRenderer` seam; the seam's block and rewake methods may need renaming once neither output is a block or a fixed line.
- The Go client today acknowledges an `AckRequired` response by itself when no handler is given, and `Client.CallWatching` resumes a `VerbHookWait` request through `resume` after a `connLostError`, which the acknowledgement's write and `awaitAckConfirmation` both return; D3's client passes a handler and keeps a ring outcome that reached it off that resume path.
- The merged Stop command, `creelStopHookCommand` in `internal/shuttleengine/claudeengine/settings.go`, runs `hook-read` only while the strand's flag stands and records the turn end after any exit other than 0; D6's echo relies on both, and the flag's source, `flaggedFilesLocked` in `internal/creelengine/flags.go`, gains the owed echo.
- `TestSmokeCreelHookDelivery` in `internal/creelcli/smoke_llm_test.go` holds the tier3 subtests D1 to D3 extend.
- The D3 criterion for the client leaving or the daemon stopping after the ring was answered races `awaitOutcome`'s select over the outcome, the connection close and the daemon's shutdown, so its test needs a seam that orders an outcome already in the waiter's channel before the close or the shutdown is taken.
