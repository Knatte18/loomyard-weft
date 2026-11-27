---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
---

## Changes

All changes are to test files.

- F1 (`cee47f0a8`): added a `TestFixerRun_Outcomes` row where the remote tip moves to `older`, an ancestor of the start head, and the run ends fixed.
- F2 (`cee47f0a8`): added `fixerRunFixture.ancestorErr` and a subtest asserting `run` wraps it and says what the query tested.
- F3 (`cee47f0a8`): added `omitSessionID` and `omitEngine` to `fakeFixerSeats`, with a row for `session: none recorded` and one for the engine-less form.
- F4 (`1de93b385`): the changed-task-branch fix review row now asserts the session.
- F5 (`f65723a7b`): `TestPushBranch_LeavesTraceLine` is now the table `TestLandedPush_LeavesTraceLine` over `PushBranch`, `DeleteRemoteBranch` and, beyond the proposal, `DeleteRemoteBranchLeased`. The former test's assertions are all kept in the `PushBranch` row.
- F5 (`66af13d40`): the test file's header now names the trace line, from the self-check.

Tier0 and tier1 passed on `internal/landingshed` and `internal/gitrepo`; `internal/gitrepo` passed again after the header change.

## Self-check

- fix-consistency, LOW, the header of `pushbranch_integration_test.go` no longer described the file: fixed.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

## Deferred

## Rulings
