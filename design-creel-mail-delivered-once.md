# Creel mail reaches the model and the operator once, in one message, and is logged

`creel-mail-delivered-once` · Task · bug, enhancement, creel · running · Drift-Verify

## Goal

One creel mail reaches the model once and the operator once, as one message, with no empty turn and no error label, and the hook command text never enters the model's context.

## Seen

After the landing of `creel-hook-delivery-fixes`, the orch session still gets every creel mail as a pair of Stop-hook messages (operator, 2026-10-29).
The mail text reaches the model once, in the hook's additional context, but the surrounding messages are doubled and carry the hook command.

One mail arrives in two steps:

1. The `hook-wait claude` ring wakes the idle strand with a system notification titled `Stop hook feedback`, whose body is a `Stop hook blocking error from command "Stop"` holding the bell line `[creel] Mail is waiting for you. End this turn now; ...`.
   The woken turn ends with nothing done, so each mail costs one extra model turn; `internal/creelengine/doc.go` states it as the ring's bound.
2. When the turn ends, the `hook-read claude` hook delivers the mail as a user turn that holds both a `Stop hook feedback:` line, `[creel] Mail for you arrived as this turn ended; ...`, and a `Stop hook blocking error from command:` message that repeats that same line after the whole shell text of the hook command, with the mail in `Stop hook additional context`.

The model reads the pointer line twice in step 2, and the full hook command text, with its paths and flag-file logic, sits in its context.

The operator's terminal shows, for one mail:

- A `Stop hook feedback` block headed by a line `Stop says: [creel] #13 from ...` that holds the whole mail text; what writes that line is not known.
- Then `Ran 2 stop hooks` with a `Stop hook error:` line holding the pointer, and a `Stop hook feedback:` line holding the whole mail text again.
- The hook command text is not shown there.

So the operator reads the mail twice, and a normal delivery is labelled an error.

## Scope

- Find why the harness shows the `hook-read` result both as feedback and as a blocking error, and whether the exit code, the `decision` field or the `reason` field of the hook's JSON causes the extra copy and the error label.
- Find what writes the `Stop says:` line and why it carries the mail text.
- Make the pointer line reach the model once, show the mail to the operator once, never label a delivery an error, and keep the command text out of the context.
- The ring hands the mail over itself: when the worker rings, `hook-wait` claims the strand's claimable mail as a boundary claim does, writes it as the rewake output in place of `BellLine`, and acknowledges the claim once the output is written, so the woken turn starts with the mail and no empty turn runs.

## Open

- Whether an asyncRewake output written by a hook that then exits is shown to the model for sure, so acknowledging on write never journals read a message the session did not see; the Stop path's block count guards the same risk there.
- The 8 KiB budget of a claim and whether the rewake output takes the same size.
- A ring that races a turn the operator starts: the mail would land under the operator's prompt, which the Stop path already does today.
- A model may trust mail in additional context less than in a block reason; the tier3 `TestSmokeCreelHookDelivery` run asked for by the earlier landing is the check.
