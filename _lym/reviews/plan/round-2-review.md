---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lym/plan/01-gate-test-map-mode.md:52
    summary: Card 1 leaves gateslot Pool.TryAcquire and testcover FuncIndex.ProfiledFuncs reached by no main, and no card deletes or allowlists them, so the test-only-code guard fails after card 1 and stays failed after card 2
    fix: >-
      In card 1's **Delete:** list add `internal/gateslot#Pool.TryAcquire`,
      `internal/gateslot#TestTryAcquire_TakesAFreeSlotAndNeverWaits` and `internal/testcover#FuncIndex.ProfiledFuncs`.
      In card 1's Intent say why each goes:
      `Pool.TryAcquire` had the audit round (`testmapaudit.go`) as its one production caller,
      and `FuncIndex.ProfiledFuncs` had `packageRun.newBlockTable`'s self-check branch as its one.
      The `Pool.TryAcquire` sentence of `internal/gateslot/doc.go` (card 1 already edits that file) goes with it.
      In card 2's Intent add that `internal/testcover/doc.go` also drops its `ProfiledFuncs` sentence.
      Overlaid on card 1 with `lym loom validate-plan --overlay`, together with the F2 change, the plan gate reports no new finding.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/plan/01-gate-test-map-mode.md:114
    summary: Card 1 passes the green record to computeScope, so a computed scope takes its diff base from tier0's green record and a clean tree after a green tier0 run maps no package, against D1's merge-base diff and D7's no-stand-in rule
    fix: >-
      In card 1's Intent paragraph beginning `runTestMap computes the scope through computeScope`,
      pass a nil green record to `computeScope` in place of `openGreen`'s result,
      and drop `internal/gatecli#openGreen` from card 1's **Uses:** (the plan gate accepts the drop).
      State that a computed scope is then the diff from the merge base with the parent branch,
      and the whole module when the worktree has no parent branch, whatever the green record holds.
      Add to the Tests part one row that pins it where a parent branch exists:
      in `TestGateTestMapInATaskWorktreeKeepsItsOwnMaps`, after a plain tier0 run has left a green record at HEAD,
      `test --wait --map` with no scope flag builds the packages of the diff from the merge base.
      Reword the sentence `the map run neither subtracts from nor records into that green record`
      to say the run reads no green record at all.
---

### [BLOCKING] Pool.TryAcquire and FuncIndex.ProfiledFuncs become test-only and no card carries them

**Location:** `_lym/plan/01-gate-test-map-mode.md:52` (the **Delete:** list), `_lym/plan/02-testcover-planning-delete.md`.

**Issue:** `PATTERN-test-only-code` and the decision record's constraint on it say every function only the retired planning, audit, wait or ordering reached goes, in `internal/gatecli` and `internal/testcover` alike.
`cmd/lym/testonlycode_linux_test.go` fails for each production function no main reaches, and an allowlist entry that matches no unreachable function fails it too.
Two functions lose their last production caller in card 1 and appear in no card:
`gateslot.Pool.TryAcquire` (`internal/gateslot/pool.go:173`), whose one non-test caller is `r.pool.TryAcquire` in `testmapaudit.go:52`, a file card 1 deletes whole,
and `testcover.FuncIndex.ProfiledFuncs` (`internal/testcover/blockkey.go:170`), whose one non-test caller is `index.ProfiledFuncs` in `packageRun.newBlockTable` at `testmappackage.go:475`, the branch card 1 drops with `blockTable.profiled`.
Only `pool_test.go` (`TestTryAcquire_TakesAFreeSlotAndNeverWaits`) calls `TryAcquire`, and no test calls `ProfiledFuncs`.
Card 1's allowlist sentence names `PlanUpkeep`, the selector, `Map.FailedAt`, `Toolchain.GoRoot` and "the helpers only they reach"; neither function is reached by those.
Card 2 deletes the selector and upkeep files and `FailedAt`, `stampDifferences`, `GoRoot`, and also does not name them.
So the guard fails at the batch gate after card 1, and fails at the end of the plan, since nothing ever removes the two functions.
This answers the focus question on cards 1 and 2: the guard does not pass at every batch boundary as written.
The fix was overlaid on card 1 and passes the plan gate, because card 1 lists no `internal/testcover` or `internal/gateslot` package or file beside these members, and its `internal/gateslot/doc.go` is a different file from `pool.go`.
Round 1's F7 added the allowlist for the selector and upkeep code; it did not reach these two, which the audit round and the self-check branch reached, not the planning.

### [MEDIUM] A computed --map scope takes its base from the green record

**Location:** `_lym/plan/01-gate-test-map-mode.md:114` and the **Uses:** entry `internal/gatecli#openGreen`.

**Issue:** D1 says the scope is chosen as for any call, naming "the diff from the merge base" for a call with no flag, and D7 says a map run subtracts no green tier "so a green record never stands in for a measurement".
Card 1 calls `computeScope` with `openGreen`'s result.
In `computeScope` the computed form reaches `diffBaseGroups` with `keptTiers` of the request, which lists tier0 alone, and `diffBaseGroups` takes tier0's base from `green.record.newestWholeAncestor` before the merge base (`scope.go:393-398`).
`openGreen` returns nil only for a dirty worktree (`green.go:272-281`).
On a clean worktree with a whole green tier0 entry at HEAD the base is HEAD, the diff is empty, `DeriveTargets` returns no module and not `Whole`, and card 1's package-set rule (the union of tier0's and tier1's module targets) is the empty set.
So `lym gate test --map` with no flag after a green tier0 run maps nothing and exits 0.
This answers the focus question on the scope outcomes computeScope can return:
the plan states the package set for `--all`, a scoper other than `go`, a computed or `--base` fallback and a whole tier share, and states that a map request names no tier so tier0's share alone decides a narrowed set (`keptTiers` and `listedTiers` confirm this: with no flag `listedTiers` returns tier0).
It misses the outcome where green steers the base.
The card-3 subtest `AMapRunWritesNoRunRecordAndAGreenRecordDoesNotStandInForIt` uses an explicit target, which bypasses `computeScope`'s diff, so it does not catch this.

## Rejected

- Fixture narrowing under `scoper: go`: `impactset.listPackages` runs `go list -tags ... ./...` with no `-test`, no `-deps` and no `-e`, and a scratch module whose test file imports an unrequired package exited 0 with the package listed, so `DeriveTargets` can narrow over the fixture; `ModuleMainPackages` uses `go list -e`, which also tolerates it.
  The `--base HEAD~1` subtest narrows to `steady` since the hub's `_lym` is a weft junction excluded from the warp index, leaving the prime clean for `ChangedPaths`.
- Fixture packages without tests: after card 1's rule `notests` and `ignored` leave the set, `dirty` and `locked` are dropped from the fixture, and `linked` and `linked/inner` stay through the hand-written maps, so every kept subtest still finds the maps it reads.
- Guards removed in cards 1 and 4: the build lock, the pre-walk status read and the per-package HEAD read each carry a stated bound in the Intent, with card 4's bound naming the unseen revert and the gitignored file.
- Card 3's memory refusal: the gate config admits `memory_mb: 64` beside `redundancy_test_memory_mb: 128` (`tiers.go` refuses only below 1), so the named input reaches it, and card 1's counter refusal is reached by a `vstest` tier the config also admits.
- Card 4's failure-while-tests-run guarded write names no separate input, which round 1's F13 resolved by the stated option that the path shares `checkClean`'s call inside `writeFailed` with the failed-build path; the card states exactly that, so it is not re-raised.
- Card 3's `TestMapSideBySide` and `TestMapSlotAsk`: round 1 rejected this as a per-symbol finding because the tables pin cases no integration subtest reaches (`Parallel` 0, a capped count below the quotient), and the re-read of `AMapRunHoldsTheLargerSlotAskOfItsTiers` and `ATiersMemoryMBClampsItsTestsToOneAtATime` agrees: neither reaches those rows, so no public-surface test covers them.
- `TestGateRedundant` wanting exit 1 from `test --wait --map cov` in card 3: the fixture's `cov` holds `TestFails`, a tier0 test that fails.
