## Changes

- F1: `TestFixVerifySpawnOrAttach` gained a table of pre-spawn refusals: no model-spec for the recovery role, a recovery engine that cannot audit forks (`noAuditEngines`), and a gate config with no `round` list.
  Each row asserts the error text, that nothing was spawned and that `st.VerifyFixers` is unchanged.
- F2: new `internal/webstercli/fixverify_test.go` drives the verb through a spawn call and an attaching call.
  Rows: done with the `reply` field and a saved status, dead with `dead_reason`, a non-zero exit and a saved status, and the no-run refusal.
- F3: `TestFixVerifyAwait` gained the dead/asking row; its `arrange` hook now receives the spawned record.
- F4: `FixVerifySpawnOrAttach` stops the strand when `FindRun` fails after `Start`, and `fix-verify` stops it when `SaveState` fails after a spawn; both join the stop error.
  `recover-batch` has the identical gap and got the same two changes.
  Tests pin the two `FindRun` cases; the two `SaveState` stops have no test, since the failure cannot be injected through the verb.
- F5: the missing-model-spec refusal now names a way forward (set the role's key, re-run the verb); a row of the F1 table pins it.
- F6: `webster-spec.md:49` says Merriam answers with a fork or the cold fixer.
- F7: `loom-plan-spec.md:309` and `:311` name the verify-gate fixers.
- F8: removed the assertions on stencil static prose in `template_test.go` and `verifyfixer_test.go`.
- F9: the `recordbatch.go` comment names the verify-gate fixers.

Fixed as proposed: F1, F2, F3, F4, F6, F7, F8, F9.
Fixed differently: F5, whose proposal offered an append or a delete; I appended a way forward in the shape of `roleAuditEngine`'s.

## Self-check

- fix-consistency, LOW: `loom-final-summary-spec.md:47` still says "after a fixer fork"; fixed as proposed.
- fix-affected-set, LOW: same site as above; merged and fixed.
- fix-affected-set, LOW: `webster-spec.md:3` names "its fixer fork"; fixed as proposed.
- fix-affected-set, LOW: `runlevel.go:1133` says only the verify-gate fixer fork runs outside a batch bracket; disputed, premise false (see Disputed).
- fix-proposal, LOW: after F8 the fork prompt row could not tell the fork prompt from the cold one; fixed with my own change, adding `testReplyPath` to the row's `notContains` instead of restoring prose assertions.
- fix-code-correctness, LOW: no test for `recover-batch`'s stop after a failed `FindRun`; fixed by `TestRecoverSpawnOrAttach_UnresolvableRunStopsStrand`.
- fix-code-correctness, LOW: no test for the stops after a failed `SaveState`; deferred (see Deferred).

## Disputed

- Premise false, fix-affected-set `runlevel.go:1133`: `forkOwnReport` maps a fork transcript to a report, and the cold fixer is a strand, not a fork, so "the verify-gate fixer fork" is correct there.
  Evidence: `forkOwnReport(st, geom, transcript)` is called only with `f.TranscriptPath` of a fork (`runlevel.go:1067`).

## Deferred

- The `SaveState`-failure stops in the `recover-batch` and `fix-verify` verbs are untested: the verb reads and saves state through package functions with no injection point, and a test that made the save fail would need a seam outside this round's surface.
