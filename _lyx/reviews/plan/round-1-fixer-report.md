# Fixer report — plan, round 1

Every finding's premise held against the cards and the code they cite (`internal/shedengine/goto.go`, `internal/shuttleengine/attach.go`, `internal/shedverbs/loop_test.go`, `contracts/specs/refusal-spec.md`, the decision record's D6).
All eleven are fixed as proposed; where a `fix` key offered two options, the one taken is named.

## Changes

- **F1** (`02-shedengine-relaunch.md`): primary option.
  `contracts/specs/refusal-spec.md` is under `**Edit:**`; `Relaunch`'s `done` and `running` messages now spell their way forward, and the missing-file refusal reuses `Goto`'s message naming `MissingStatusWayForward`.
  Added shed rows `relaunch on a done run` and `relaunch on a running run` quoting those ways forward, widened the `status file missing` row's trigger to name `step --arbiter-attempt`, and added a `TestRelaunch` row for an absent status file (`Admit` not called, error naming the way forward).
- **F2** (`14-driver-arbiter-branch.md`): the wake bullet now re-runs the arbiter command only when the driver's last action at the current stop was that command and its job ended without an envelope the driver holds (compaction, revive mid-arbiter), never on the loop job's own exit; a refusal from that re-run falls back to `## Stopping on a non-running state` or `## Error and interrupted stops`.
  The hand-on bullet excludes the wake re-run's refusal, and `wordingClaims` gains `never on the wake of the loop job's own exit`.
- **F3** (`12-relaunch-consume-attempt.md`): added an unparseable-envelope row (the parse refusal moved from the verdict to the envelope under F5) and rows `0`, `-1` and `x` driving `loomConsumeAttempt` on a `loomCLI` with a temp location.
- **F4** (`10-arbiter-verb.md`): the spawn-path table gains a missing-status-file row and an unreadable-envelope-path row, each asserting the way forward, no spawn and no attempt directory.
- **F5** (`05-loom-arbiter-records.md`, `10-arbiter-verb.md`, `12-relaunch-consume-attempt.md`): card 5 declares `ArbiterEnvelope`, `WriteArbiterEnvelope` and `ReadArbiterEnvelope` with an envelope round-trip, absence and unparseable row; card 10 writes `envelope.json` through the writer and lists it under `**Uses:**`; card 12 lists the reader, reads `envelope.json` instead of `verdict.yaml`, refuses unless it is a `resume` with an empty `cause`, takes the goto target from it, and adds the row with `verdict.yaml` `resume` under an envelope `hand-on`/`precondition`.
  The refusal-spec row text for `attempt unusable` follows.
- **F6** (`10-arbiter-verb.md`): step 2 splits the marker case: a passed deadline calls `StopStrand` and reads `hand-on`/`timeout` without attaching; otherwise `AttachIfLive` runs with a positive `Timeout`, and its error reads `hand-on`/`died` after `StopStrand`.
  The re-run table gains both rows.
- **F7** (`14-driver-arbiter-branch.md`): `**Intent:**` gains the proposed `Bound:` sentence.
- **F8** (`09-arbiter-eligibility.md`, `10-arbiter-verb.md`): first option.
  Card 9 drops `arbitereligibility_test.go` and its test-budget edit; card 10's spawn-path table now carries each eligible and refused kind, the Publish halt with and without a target, and the `awaiting` stop with a `parent_notice` but no escalation.
- **F9** (`12-relaunch-consume-attempt.md`): success rows assert the applied mark and current producer only, refusal rows the message and absent mark; pause preservation and the unchanged file are left to `TestRelaunch`.
- **F10** (`03-shedverbs-arbiter-attempt-flag.md`): the nil-hook, busy and other-error rows move into `TestLoop_RefusalsBeforeAnyStep` (now under `**Edit:**`, with an arguments-and-hook column for the waiter path); the new test keeps the flag-without-`--until-stop` row and the success row.
- **F11** (`02-shedengine-relaunch.md`): second option.
  `Goto` keeps its inline unknown-target check before the running check and calls `CheckGotoTarget` after it in place of the admission check, so precedence and messages stay unchanged.

## Disputed

None.

## Deferred

None.
