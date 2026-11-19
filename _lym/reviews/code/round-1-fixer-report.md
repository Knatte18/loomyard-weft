---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The proposed check-then-Store leaves a window where a clear runs between the check and the Store, so the stale answer survives. The fix stores first and deletes the key when the clear counter changed since before the read, which closes that window."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
---

## Changes

- F1: `internal/fabricengine/onerepository_cache_test.go` adds `TestOneRepositoryAt_KeepsAnswerPerWorktree`, a serial tier1 table with the three rows the review names.
  It is serial because every marker commit clears the process-wide answer map.
- F2: `onerepository.go` adds the `layoutClears` counter, which `forgetLayouts` increments before it clears.
  `oneRepositoryAt` reads the counter before the git read, stores the answer, and deletes the key when the counter changed.
  Fixed differently from the proposal, as the frontmatter reason says.
- F3: the `forgetLayouts` doc and the `doc.go` layout paragraph name the writes that add or retire a marker.
- F4: the `layoutAnswers` doc and the `doc.go` layout paragraph state the process bound.
- F5: the `doc.go` probe paragraph, the `MergeStateActive` doc and the comment of the cherry-pick step in `wiring_commitstatus_integration_test.go` say a weft cherry-pick, revert or rebase does not count.
- Self-check follow-ups: the `MergeStateActive` doc sentence is split into one sentence per line, so no line wraps mid-sentence or reaches 120 characters (a gate repair), and the `doc.go` sentence no longer claims a marker rewrite leaves the answers, since a rewrite through `commitAnchorMarker` clears them.

Commits: F1 and the docs and the F2 change landed in three commits, with F2 through F5 grouped because they share files.

## Self-check

- fix-code-correctness, LOW, the check-then-Store window stays open: fixed differently (the F2 change above).
- fix-code-correctness, LOW, the `MergeStateActive` doc line passes 120 characters: fixed.
- fix-consistency, LOW, the loomcli test comment still calls the cherry-pick fabric's replay: fixed.
- fix-consistency, LOW, the `layoutAnswers` doc says "marker writes" without the add-or-retire bound: fixed.
- fix-proposal, LOW, the `doc.go` rewrite clause is wider than the code: fixed by dropping the clause.
- fix-proposal, LOW, the `MergeStateActive` doc line length: duplicate of the correctness finding, fixed.
- fix-affected-set: no findings.

## Gate repairs

- The first gate run flagged a comment that wrapped mid-sentence in `mergestateactive.go`, fixed as described above.
- The next gate run listed reach reports only.
  Its two unreached symbols, `seedOneRepositoryBoard` and `MergeStateOperation`, are code this round did not change or that tests of `internal/fabricengine` and `internal/loomcli` already pin.
  No test was added and nothing else changed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
