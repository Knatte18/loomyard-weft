## Changes

All four findings are fixed.
They share files and overlap in `remoteteardown.go`, so they went in one commit (`fix F1, F2, F3, F4`) and the self-check fixes in a second.

- **F1 — fixed differently.**
  Proposed: a Topology field carrying the child command.
  Not made: `holdTeardown` takes the in-process branch whenever `testing.Testing()` is true, so a Topology field is unreachable from a test.
  Instead `spawnTeardownHold` takes the executable and its arguments as parameters, and `holdTeardown` passes `os.Executable()` and `teardownChildArgs`.
  `TestSpawnTeardownHold` (tier 1, `remoteteardown_childhold_integration_test.go`) re-execs the test binary as the child through `TestMain` and has rows for held, busy with try, busy without try, an error line and a closed stdout.
  The held row asserts the recorded lock file and the `RoleTeardown` record under the slug.
  The existing `childHold.finish` rows stay.
- **F2 — fixed as proposed.**
  `NoticeRemoteTeardownEnded(hubPath, slug, reason)` is exported from `remoteteardown.go`.
  `warnWhenEnded` (now taking the hub path) and `childHold.finish` on a failed job write both call it, replacing their bare Warn.
  `TestNoticeRemoteTeardownEnded` has a row for each reason, and `TestChildHold_Finish_NoticesAnUnwritableJob` pins the `finish` wiring.
  The signal path itself has no test: it ends the process through `os.Exit`.
- **F3 — fixed as proposed.**
  `noticeUnreadRemoteTeardown` and the ended notice share `unknownKindWayForward`, which names the discard alternative.
  `TestRunTeardownChild_NoticesAnUnreadableJob` asserts all three alternatives.
- **F4 — fixed as proposed, with one addition.**
  `failedRemoteSteps` and `noticeRemoteTeardown` take the records repository's root; `RunTeardownChild` looks it up only after a failed step, and an unresolved root drops the manual tag push and leaves the re-run.
  A records-repository lookup failure after a successful archive is its own step ("the lookup of its records repository") through `remoteOutcome.weftRootErr`.
  `TestNoticeRemoteTeardown` has rows for the unresolved root and the lookup failure.
- **Self-check fix.**
  `runRemoteTeardown` also keeps the lookup failure in `out.err`, so the in-process callers (`remove.go`, `cleanup.go`, `taskbranchdiscard.go`) still see a failed removal; `failedRemoteSteps` checks `weftRootErr` first.
  `RunTeardownChild` now mails the notice when it cannot answer its parent.

Gate: `lym gate test ./internal/fabricengine ./internal/fabriccli --tier 1` passed, 0 failed.

- **Gate repair.**
  The header comment of `remoteteardown_childhold_integration_test.go` was fixed-column wrapped; it is now one sentence per line.
  It is a comment-only change, so the tier-1 result above stands.

## Self-check

- fix-code-correctness — no findings.
- fix-consistency — MEDIUM, a lookup failure dropped from the in-process verb result: fixed as proposed (combined error kept in `out.err`).
- fix-proposal — MEDIUM, the same dropped lookup failure: duplicate of the fix-consistency finding, fixed with it.
- fix-affected-set — LOW, `RunTeardownChild` returning on a failed parent answer with no notice: fixed as proposed (calls `NoticeRemoteTeardownEnded`).
- Not tested: the in-process consequence of `weftRootErr` (a real lookup failure after a successful archive needs a hub fixture).
  `RecordsRepositoryLookupFailure` pins the reporting side only.

## Disputed

None.

## Deferred

None.
