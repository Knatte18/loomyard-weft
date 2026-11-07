# Fixer report — round 1, code

All seven findings fixed; the gate (`lym gate test ./internal/shuttleengine ./internal/creelcli`) is green.

## Changes

- **F1** (fixed with my own error type): `clearBoxBeforeCommand` in `internal/shuttleengine/send.go` now reads the box and checks operator activity right before `C-u`.
  Text in the box or an active operator returns a `*SessionBusyError` wrapping `ErrSessionBusy` with no key and no paste.
  A blank or unreadable box still gets the clear.
  Proposed change not made: returning an error wrapping `ErrForeignBoxText` for a box with text.
  `mapDeliveryError` (`internal/creelcli/delivery.go:464`) maps `ErrForeignBoxText` to `ErrNotLanded`, which creel does not requeue, so a command held before anything was typed would be dropped.
  A `*SessionBusyError` with `Draft: true` maps to `ErrBusy` and is requeued, matching the idle wait's own draft hold.
  Rows added to `checkPastedSessionCommand`: a draft typed after the idle reading, an operator active only at the clear, and a box unreadable at the clear.
- **F2** (fixed with my own matcher): `confirmPasted` fails a command with `ErrCommandTakenAsPrompt` only on a turn start past the paste whose instruction is empty or the command's text, through the new `commandTurnStartedSince`.
  Proposed change not made: `promptTurnStartedSince(..., text)`, which matches the instruction against the text alone.
  `typedInstruction` (`internal/shuttleengine/claudeengine/signals.go:209-220`) records an empty instruction for a slash command, so that match never fires on the real engine and a command taken as a prompt would be reported as sent.
  An operator's typed prompt carries a non-empty instruction and no longer fails the command.
  Rows: an empty-instruction turn start fails with `ErrCommandTakenAsPrompt`, and a turn start with another instruction leaves the command confirmed.
  `slashPasteEngine` in the test fakes now parses `START:<text>` as a turn start with that instruction.
- **F3**: a `ErrCommandTakenAsPrompt` row in the `Command` table of `internal/creelcli/delivery_test.go`, expecting the same error out.
- **F4**: rows in `checkPastedSessionCommand` for a failed clear key (error, no paste) and for an engine without `InputBoxClearer` (one paste, no key).
  A new `clearlessPasteEngine` and a `pasteEngineShape` parameter on `newSlashPasteRunnerAppending` carry the second row.
- **F5**: one sentence each in `internal/creelengine/doc.go` and `internal/creelcli/doc.go`.
- **F6**: `docs/overview.md` says launch, resume and open.
- **F7**: both TTL comments in `internal/shuttleengine/claudeengine/template.yaml` say a run starts or a session opens.
- Doc comments of `sendContext`, `confirmPasted`, `clearBoxBeforeCommand`, `ErrCommandTakenAsPrompt` and `internal/shuttleengine/doc.go` follow the new behaviour.

Commits: one per finding, except F1, F2 and F4, which share `send.go` and `session_test.go` and went in one commit, with a follow-up commit for F2's matcher.

## Self-check

- fix-code-correctness, BLOCKING: `promptTurnStartedSince` never matches a slash command's empty instruction. Fixed with my own change (`commandTurnStartedSince`).
- fix-code-correctness, LOW: no `claudeengine`-level test covers the match. Not fixed: the match runs in `shuttleengine` over the engine's signals, and the empty instruction it relies on is `typedInstruction`'s existing, documented behaviour.
- fix-proposal, BLOCKING: the same finding as fix-code-correctness's. Merged; fixed.
- fix-consistency, MEDIUM: five comment lines of 120 characters or more. Fixed by re-wrapping at clause boundaries.
- fix-consistency, LOW: `ErrCommandTakenAsPrompt` doc stale against the match rule. Fixed.
- fix-affected-set, LOW: the same stale doc. Merged; fixed.

## Gate repair

The gate refused the round's comments for wrapped breaks and over-long lines, before any test ran.
Nine comments were rewritten one sentence per line in `internal/creelcli/doc.go`, `internal/creelengine/doc.go`, `internal/shuttleengine/doc.go`,
`internal/shuttleengine/send.go` and `internal/shuttleengine/session_test.go`.
No code changed in the repair.

## Disputed

None.

## Deferred

None.
