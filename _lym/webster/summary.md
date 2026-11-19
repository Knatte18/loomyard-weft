# One-repository layout in fabric: all ten batches done

Every batch recorded `status: done`. No digest listed departures or renames, and none carried a warning.
The scope guard listed deviations (files changed outside a batch's own targets) on every batch except 15.
Batch 04's deviations include `internal/gitrepo/push.go`; batch 08's include `internal/gitrepo/merge.go`.
The other deviations are test-guard allowlist files, new production or test files in `fabricengine`, and small wiring files in other packages.

- **01 lymcwd-one-repository** (head `94a81a57`): cards 1 to 3.
  `lymcwd` reads the anchor from the worktree root, and fabric gained the records paths, the commit predicates and the one-repository geometry.
  Deviations: the two `cmd/lym` guard tests, `cleanupremotewarp.go`, `onerepository.go`, `recordspaths.go` with their tests, and `lymcwd.go`.
- **04 protected-push-class** (head `559899b3`): the protected-branch match moved into fabric, with a recorded `protected` failed-push class.
  Deviations: `testonlycode_linux_test.go`, `pushclasses_integration_test.go`, `pushclassified.go` and `gitrepo/push.go`.
- **05 one-repository-clone** (head `5a1a72ec`): clone by one URL or `--single-repo`.
  Deviations: `uncontainedwrite_test.go`, the board and fabriccli integration tests, and `clone.go`, `clone_onerepository.go`, `junction.go` and `recordsgit.go` in `fabricengine`.
- **06 one-repository-add** (head `fe9d0cb8`): cards 6 and 7, Add without a sibling and the path-limited records commit with `index.lock` retry.
  Deviations: `testonlycode_linux_test.go`, `forkrecords.go`, `recordscommit.go` and two integration tests.
- **08 one-repository-anchor-change** (head `a72537a0`): cards 8 and 9, the anchor change as one trailer-carrying commit and merges that restore the receiving side's records.
  Deviations: 15 files, mostly in `fabricengine`, including `destroy.go`, `gitrepo/merge.go` and two `cmd/lym` guard tests.
- **10 records-commit-during-merge** (head `c4877cb5`): records commits around a merge, abort and continue of an unsettled merge, and `MergeStateOperation`.
  Deviations: `destructiveguard_test.go`, `midmerge.go`, `recordsduringmerge.go` with its test, and `loomcli/start_midmerge_test.go`.
- **11 transition-seam-waits** (head `f811661d`): `statuscommit` waits on a blocking operation and completes on a protected-only push failure.
  Deviations: `testonlycode_linux_test.go`, `battencli/overview_test.go`, `loomcli/wiring_commitstatus_integration_test.go`, `statuscommit.go` and its test.
- **12 one-branch-push-and-teardown** (head `13cf5931`): cards 12 and 13, one-branch push and teardown, and the records-only replay.
  Deviations: 13 files, `testonlycode_linux_test.go` and `fabricengine` files for push, merge, pull, replay, teardown and takeover.
- **14 prime-records-push-guard** (head `d20b6b80`): the guard on the prime's records push, owed as `operator_unpushed`.
  Deviations: `primepushguard.go` with its test, `pushclassified.go`, and `pairteardown/runread.go` with its test.
- **15 one-repository-docs** (head `a050224c`): cards 15 and 16, the docs and the PATTERN changes.
  No deviations.

## Verify gate findings

The plan-level verify failed on one test:
`internal/landingshed.TestFinalize_OverRealHub/resolves_a_conflict_and_squash_merges_into_the_parent`.
A fixer then ran.

## Open points

These come from the fork reports, outside the digests.

- The pre-push origin reconcile blocked on a remote-refs lock in a one-repository hub, so batch 11's new rows skip that check.
- `Topology.Reconcile` still calls `retireBoltAnchorMarker` in a one-repository hub, and that can delete the board's marker. It sits in `anchormarker.go`, which no card targeted.
- Two `TestReplay_OneRepository` rows were left out, `ReconcileReportsTheBranchDiverged` and `OriginAtAnotherAnchorIntegrates`.


## Integration suite triage

The `final` gate failed, but webster's triage did not attribute the failure to this run.

Flaky (passed on rerun):

- `github.com/Knatte18/loomyard/internal/reedcli`


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `a823f323871d529b754913dff19db45bf3b4fcb2` fix: expect the Fabric-Merge trailer on the landing commit message
