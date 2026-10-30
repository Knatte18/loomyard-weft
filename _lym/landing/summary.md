# Deliver creel mail once to model and operator, and log each delivery

Closes #900

## What changes and why

A creel mail used to reach an orch session as a pair of Stop-hook messages.
The idle ring woke the strand with a blocking-error notification that held only the bell line, so each mail cost an empty model turn.
The Stop delivery then repeated the pointer line, once as feedback and once as a blocking error that carried the whole hook command text into the model's context.
The operator read the mail twice, and a normal delivery was labelled an error.

After this change one mail reaches the model once and the operator once, with no empty turn, no error label and no command text in the model's context.

- A Stop delivery by `hook-read` prints `hookSpecificOutput.additionalContext` alone, with no `decision`, `reason` or error label, and exits 0.
- A tool-boundary delivery prints the lines as `additionalContext` plus a `systemMessage` for the operator, never a block.
- The idle ring hands the mail over itself.
  A `hook-wait` request carries a mark that its waiter takes mail, and the daemon claims for a ring only to a marked waiter.
  `hook-wait` writes the lines as the rewake output on stderr, acknowledges, and exits with the wake code, so the woken turn starts with the mail.
  `creelengine.BellLine` is no longer its output.
- The waiter's Stop hook sets `rewakeMessage` (`[creel] Mail that arrived while you were idle:`) and `rewakeSummary` (`creel mail`), so the wake carries creel's labels instead of Claude Code's blocking-error default.
- Claude Code draws the wake as a single row and never its body, so the ring's mail reaches the operator as an echo.
  The daemon keeps the lines of an acknowledged ring in memory, and the strand's next claim hands them out ahead of any mail.
  `hook-read` renders the echo as a `systemMessage` under `[creel] Delivered while idle:`.
- The daemon logs one line per claim, on the tool, stop and ring paths, with the strand, ids, echo ids and outcome (`acknowledged`, `released`, `ack-failed`, or the reason nothing was handed out), and one line per bell send.
  The levels are `Info` for a claim that hands out mail or an echo, a release and a bell, `Warn` for `ack-failed`, and `Debug` for a claim that hands out nothing.

The package docs and comments that stated the block design are rewritten: `internal/creelengine/doc.go`, `internal/creelcli/doc.go`, `internal/shuttleengine/claudeengine/doc.go`, the ring outcome in `internal/creelengine/pool.go`, and the orch stencil.

## Design decisions

- All Claude-specific shapes (the JSON keys, the rewake text, the `rewake*` keys) stay in `internal/shuttleengine/claudeengine` behind the `ToolBoundaryContextRenderer` seam.
  `hook-read` and `hook-wait` in `internal/creelcli` pass the event or the lines to it.
- The ring claims under the boundary rules: pause read through the seam before the locked step, hold, `hookFieldCap`, the 8 KiB budget with the first message always handed out.
  The claim stays reserved until the client acknowledges on the same connection, and every way the answer path ends before the journaled acknowledgement releases it.
  A ring whose claim comes back empty does not ring and the waiter stays registered.
- Once a ring carrying mail has reached its handler, `hook-wait` never resumes the wait through the client's reconnect path, so a connection lost at the acknowledgement ends the call.
- A journal write that fails at an acknowledgement now releases every id it left unjournaled, where `handOut` used to leave the ids after the failing one reserved.
  This trades a loss for a duplicate: the mail the hook already showed is dropped, and the mail the daemon did not journal reaches the model once more at a later claim.
- A Stop claim that hands out only an echo prints the `systemMessage` and exits 1, so the merged Stop command still records the turn end.
- The waiter's mark is a request field an older daemon ignores; an unmarked waiter is rung with no mail as before.

## Known bounds

- The ring's mail is journaled read before Claude Code shows it, so a session that dies in that window loses it as read; the `acknowledged` log line records it.
- A restart of the daemon drops an owed echo, so the operator then never sees that ring's mail, while the model's copy is unaffected.
- The echo shows only at the woken turn's first claim; a turn an API error ends fires `StopFailure`, which claims nothing.
- The Stop summary row clamps to 5 lines, so a claim longer than that shows the operator only its head there.
- Sessions launched before this change keep the default wake labels until relaunched.
- The Stop and wake behaviour is proven for Claude Code 2.1.312, and the `@internal` `rewake*` keys are not a public contract.

## To check by hand

- Run the tier3 `TestSmokeCreelHookDelivery` smoke, extended here but not run in this session, and rerun it after each Claude Code update.
  It checks that a real session's model acts on additional-context mail, that the transcript holds no hook command text or `hook blocking error`, and that an idle strand wakes with the mail in its first message and shows the operator one `creel mail` row and the mail once.
- A hook-wait subtest superseded by the new ring behaviour may remain in the smoke or CLI tests; check that none still expects the bell line as the wake.
- The D3 bounds and the delivered-once exceptions (mail shown whatever the acknowledgement's outcome, mail lost in the read-before-show window) must be stated in the docs; check `internal/creelengine/doc.go` states them.
- The orch stencil, `contracts/stencils/orch/orch-template-role.md`, must not still describe the bell-first wake with an empty turn; check its wording against the new ring.
- A model may weigh mail in additional context less than a block reason; check the smoke shows it acting on the `[creel]` lines.
