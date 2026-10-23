## Changes

- F1 (fixed as proposed, one adjustment): `reclaimEntryTimeStrands` now sets a stopped cold fixer's record to dead/died and returns `stateChanged`; `Run` saves `st` right after the reclaim only when a record changed.
  The proposal's "add a SaveState if missing" was narrowed to the changed case, because an unconditional save broke tests that make the state dir read-only.
  The run-entry reclaim test asserts the saved terminal record stays done and the stopped one is dead/died.
  It does not drive `FixVerifySpawnOrAttach` end to end: that spawn-after-dead rule is already pinned by the dead-last-record case in `verifyfixer_test.go`.
- F2 (fixed as proposed): the shared fixer body says a write under the plan directory or `_lyx` corrupts the run's own record; `doc.go`'s cold-fixer section states the run-exit audit reads Master's forks only.
- F3 (fixed as proposed): `writerRecovery` reads "a recovery or verify-fixer session"; the `RunWrites.Recoveries` comment and the pinning rows in `recoverbatch_test.go` follow.
- F4 (fixed as proposed): the cold-fixer prompt row pins `testFixGate` alone.
- Self-check follow-ups: the aggregate clause "a fork or a recovery session wrote last" is now "a fork, recovery or verify-fixer session wrote last" in code, tests, `doc.go` files and the accept-audit help text; the `contractFileStatus` comments, the `doc.go` write-evidence session lists and the cold-fixer lifecycle sentence name the fixer sessions and the dead/died save.

## Self-check

- fix-code-correctness, LOW: the reclaim test does not drive `FixVerifySpawnOrAttach`; fixed with own change (the saved record is asserted, the attach rule stays pinned in `verifyfixer_test.go`).
- fix-proposal, LOW: aggregate "wrote last" phrase left unchanged beside the new label; fixed (widened at every site).
- fix-proposal, LOW: `doc.go` omits the reclaim's dead/died save; fixed.
- fix-affected-set, LOW: aggregate "wrote last" phrase, duplicate of the fix-proposal finding; fixed.
- fix-consistency, LOW: `doc.go` omits the dead/died save; duplicate, fixed.
- fix-consistency, LOW: `contractFileStatus` comment says "recovery write"; fixed.
- fix-consistency, LOW: aggregate "wrote last" phrase; duplicate, fixed.
- fix-consistency, LOW: `doc.go` write-evidence session lists omit the cold fixer sessions; fixed.

## Disputed

None.

## Deferred

None.
