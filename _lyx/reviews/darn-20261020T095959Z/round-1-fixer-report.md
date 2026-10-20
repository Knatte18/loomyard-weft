# Fixer-report, round 1

## Changes

- F1, fixed as proposed (`a13736fe`): `TestStorePrebuilt` gained a row for a pre-published store entry under the content name and a row for a store path that is a regular file, and every row now asserts no `.partial-` staging file remains.
  The pre-published row reaches only the reuse path, since `storePrebuilt` finds the entry before it copies; the self-check showed the lost-race branch needs a call to `copyIntoPlace` itself, covered under the self-check changes below.
- F2, fixed as proposed (`d8e0b823`): `storePrebuilt` now falls through to `copyIntoPlace` when `os.Chtimes` on an existing entry returns `os.ErrNotExist`.
- F3, fixed as proposed (`fd687569`): a new subtest of the tagged-run integration test runs a tagged gate from a directory outside any hub and expects the exported prebuilt path to be the build's own `-o` path, with its directory removed afterwards.
- F4, fixed as proposed (`4c327226`): `internal/shedverbs/loop.go` uses `logger.TraceIDEnv` at both sites; `shedverbs` already imported `internal/logger`.
- Self-check changes (`919e62db`):
  - Two subtests inside `TestStorePrebuilt` call `copyIntoPlace` directly: a destination that cannot be replaced is kept and no staging file remains, and a missing source returns an error and leaves nothing behind.
  - `logger.TraceIDEnv` replaces the literal in `internal/reedengine/lifecycle.go` and `internal/shedverbs/loop_test.go`.
  - Gate repair: the same replacement in `internal/testkit/tmuxkit/tmuxkit.go` was reverted.
    `tmuxkit` importing `logger` closes an import cycle in `internal/dotgit`'s tests (`dotgit` tests import `tmuxkit`, which then reaches `lyxcwd` and back to `dotgit`), which failed the verify run.
    The literal stays there, the same constraint the file's own comment states for the engines' scrub.
  - The `storePrebuilt` doc comment states that an entry pruned before its refresh is copied in again, and the `prebuild_test.go` header lists the new coverage.

## Self-check

- fix-consistency, LOW, `loop_test.go` literal: fixed as proposed.
- fix-consistency, LOW, `storePrebuilt` doc comment stale after F2: fixed as proposed.
- fix-consistency, LOW, `prebuild_test.go` header stale after F1: fixed as proposed.
- fix-proposal, MEDIUM, F1's pre-published row does not reach the rename-lost-race branch: fixed with my own change, the direct `copyIntoPlace` subtests; the finding's suggested non-empty-directory destination is used there, since through `storePrebuilt` it would take the reuse path.
- fix-proposal, LOW, the staging-file glob is weak where the rename succeeds: fixed through the same subtests, where the lost-race case makes the glob assertion meaningful.
- fix-code-correctness, LOW, same as the fix-proposal MEDIUM finding: merged and fixed with it.
- fix-affected-set, LOW, `reedengine/lifecycle.go` literal: fixed as proposed.
- fix-affected-set, LOW, `tmuxkit/tmuxkit.go` literal: fixed as proposed, then reverted after the gate failed; see the gate repair above.
- fix-affected-set, other test files that spell the literal (`reedengine/lifecycle_test.go`, `shuttleengines/shuttleengines_test.go`, `tmuxkit/tmuxkit_integration_test.go`): not changed; the fork rated them lower and the review's fix key did not reach them.
  Listed under deferred.
- No fork returned without a report.

## Disputed

None.

## Deferred

- The literal `LYX_TRACE_ID` remains in `internal/testkit/tmuxkit/tmuxkit.go`, where `logger` cannot be imported without an import cycle, and in `internal/reedengine/lifecycle_test.go`, `internal/shuttleengines/shuttleengines_test.go` and `internal/testkit/tmuxkit/tmuxkit_integration_test.go`.
  Those are test fixtures outside what F4's fix key and premise named, and the integration-tagged ones were not run this round.
