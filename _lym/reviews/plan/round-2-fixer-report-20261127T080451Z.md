---
dispositions:
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed-differently
    reason: "Carried out as proposed except the multi-store walk: the proposal's stop at the first failing store let an unreachable code origin hide a reachable records origin on every re-request (self-check, fix-proposal lens). The walk now reads every held store in the fixed order and returns the first failing store's error."
---

# Round 2 fixer-report

## Changes

- **F15** (fixed):
  - `00-overview.md`, decision "the foundation's tests shrink":
    - Cut the clause that had the implementer stop at card 15 and ask the parent. Card 15's commit body still reports the measured totals.
    - Added the estimate: about 1,050 added `_test.go` lines, file headers, `TestMain`s and `export_test.go` accessors included. Against that, about 530 deleted by cards 2, 3, 4 and `TestList_RefStorage`, and about 820 more deleted by card 15, so the part ends about 300 lines down.
    - The deletion rule now also names card 15 as the home of a deletion no other card touches.
  - Card Index line 15 names the test deletions.
  - `15-snapshot-reader.md`:
    - Adds 13 whole-test and helper deletions and 21 partial test edits in `internal/fabricengine`. Each names the test that holds its assertion, or why it guards no rule of its own.
    - The candidates came from a search of the package's tests, and the self-check forks verified the holders.
    - `lym loom validate-plan --overlay` passes.
- **F16** (fixed differently, see the frontmatter):
  - Card 15 fixes the store walk order as `StoreCode`, `StoreRecords`, `StoreBolt`, skipping stores the reader does not hold. A failing store no longer stops the walk; the error is the first failing store's.
  - Card 15 names each field's error `Store`: `FieldWorktrees` carries `StoreCode`, `FieldAnchors` and `FieldRecordsStore` carry `StoreRecords`, and `FieldPendingTeardowns` carries the zero `Store`.
  - Card 14: a phase enters `Phases` only after its read succeeds.
  - Card 14: "latest phase" is the phase requested last, whether or not its read succeeded.
  - Card 14: a re-read's `ReadFailure` records the phase the re-read ran under.
  - Card 14: `TestRunRule`'s transport row asserts `Phases` holds the phase once.
- Items from the review's "Seen but not admissible" list that I fixed, since they lie inside this surface:
  - Card 15: `TestSnapshotReader_RefStorage` reads `FieldLocalRefs` from the linked worktree's root. The subdirectory read is kept for `FieldWorktrees` alone, because `gitrepo`'s open refuses a non-repository path (`internal/gitrepo/gogit.go:58-60`).
  - Card 2: `TestStructuralDirs` overwrites the returned slice's first element as well as appending to it.
  - Card 3: `TestIsReservedHubName` asserts `HubReservedNames()` equals its literal members, instead of using predicate rows.
  - Card 13: `TestBuilder`'s second row overwrites the built snapshot's slice elements and wants the next `Build` unchanged.
- Left for the parent, not changed: card 4's seed and sync subject rows of `TestSeedCommitSignature`, and card 6's callers of `Snapshot` and `Extend` on a record value (`internal/pairteardown/teardown.go:582-583`, `internal/fabricengine/originreconcileended.go:125`, `internal/fabricengine/originreconcile.go:815`). Each needs reading of code outside this round's findings.

## Self-check

- fix-consistency, MEDIUM, "latest phase" ambiguous once `Phases` records only successful reads: fixed (card 14 defines it as the phase requested last).
- fix-consistency, LOW, overview deletion rule says "in the card that touches it": fixed (names card 15 for untouched files).
- fix-consistency, LOW, Card Index line 15 stale: fixed.
- fix-claims, MEDIUM, `TestMergeState_Record` holder misnamed: fixed (`TestMergeSiblings_CommitRefusesForeignMergeState`).
- fix-claims, MEDIUM, `TestRemove_ForceStillArchives`' holder does not pin the tag name or target: fixed by keeping the test, so the estimate drops to about 820.
- fix-proposal, MEDIUM, stopping at the first failing store hides a reachable records origin: fixed with my own change (the walk reads every store and returns the first failure).
- fix-affected-set, LOW, `fabric_test.go` header names the deleted test: fixed (card 15 says the header names only the tests that remain).
  - The validator refuses a `_test.go` file under Edit, so the header edit rides on the `TestNew_HappyPath` deletion and has no target line of its own.

## Disputed

None.

## Deferred

None.

## Rulings

None.
