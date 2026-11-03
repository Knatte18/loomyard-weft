# Fixer-report — code review round 1

Gate: `lym gate test --tier 0 --tier 1` over `./internal/fabricengine`, `./internal/battencli`, `./internal/loomcli` and `./internal/fabriccli` passes after the last change.

## Changes

- **F1** (MEDIUM): fixed differently from the proposal.
  - Proposed and not made: adding `!entry.Prunable &&` to the `registered` predicate in `originTargets`.
    That makes every run-scope caller without `PairMayBeAbsent` (`fabriccli/remove.go`, `DropLocal` in `destroy.go`, `loomcli/landingdeps.go`, `loomcli/ownercheck.go`, `battencli/wire.go`, `battencli/resumereconcile.go`) fail with "no pair worktree" on a stale registration, where they proceeded before.
    The review's "still returns today's error" does not hold for that case: no error existed for a prunable pair.
  - Made instead: `originTargets` looks the entry up by index and sets `absentPair` when the entry is unregistered, or prunable with `scope.PairMayBeAbsent` set.
    A prunable pair then takes the absent case (slug joins `pairs`, `wanted` rejects every entry) only for a scope that may find its pair absent; every other scope keeps its previous behaviour.
  - `originTargets`, `ReconcileScope.PairMayBeAbsent` and the `doc.go` paragraph on the run scope now state the stale-registration case.
  - Test: the `PairMayBeAbsent` subtest in `originreconcile_integration_test.go` gains a row that deletes the task worktree directory without pruning and asserts `PairAbsent` and `owned_elsewhere`, plus an assertion that a scope without the option still reconciles a stale registration without error.
- **F2** (LOW): fixed as proposed, with one extra step.
  - `archiveTagName` is now `ArchiveTagName` and `archiveTagSHALen` is `ArchiveTagSHALen`, renamed in `archive.go`, `snapshotuncommitted.go` and `replay.go`.
  - `battencli/readoptstop.go` drops `archiveTagTipLength`, guards on `fabricengine.ArchiveTagSHALen` and forms the tag with `fabricengine.ArchiveTagName`; `owner_readopt_test.go` uses the exported constant.
  - `destroy.go`'s hand-formed twin of the tag name now calls `ArchiveTagName`.
- **F3** (LOW): fixed as proposed.
  A subtest with one row per skipped check (detached HEAD, empty anchor marker, standing anchor move record) runs the adopt-only Add from a spoiled prime and asserts the pair is re-made.
- **F4** (LOW): fixed as proposed.
  - `cleanupremotewarp_integration_test.go`: a lock-directory-blocked row asserts the candidate's `Error` is set, `Reason` empty, branch kept.
  - `continue_pass_test.go`: a `goneErrors` row asserts the refusal "whether its task worktree is gone does not read".
  - `landingdeps_integration_test.go`: a new test asserts `RemoveFollowUp` returns "" over a hub with no readable prime.

Commits: one per finding, plus one follow-up for F1 after the self-check.

## Self-check

- fix-code-correctness, MEDIUM: a no-option run scope errors on a prunable pair where it proceeded before — fixed with my own change (see F1).
- fix-consistency, LOW: `doc.go` and the `PairMayBeAbsent` field doc did not state the stale-registration case — fixed as proposed.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
