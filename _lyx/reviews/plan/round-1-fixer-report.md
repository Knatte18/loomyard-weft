# Fixer report — plan, round 1

## Changes

Every finding was fixed as proposed.

- F1: card 1's Intent adds the weft-root check: immediately before its first `os.MkdirAll` or `os.Rename`, the step `os.Stat`s `RecordsWorktreePath(l, slug)` and returns an empty outcome when it is absent.
  `TestWireJunctions_ScratchLinkRefusedAfterClassify` gains a row whose `SetScratchPathTrackedForTest` fn removes the pair's weft worktree and returns false, asserting no target and no root link.
  Card 2's Intent replaces "no guard of its own is added" with a pointer to that check.
- F2: card 1's Intent states that every left report whose way forward moves content into the target names the `scratchLinkTarget` path.
  The tracked leave, the case-4 shapes and every case-6 report (step 2's failed tracked check and step 3's) now name `<target>`.
- F3: card 1's Intent names the rewrite of `Unwire`'s doc comment and `Topology.Remove`'s sweep comment, and card 2's Intent names the rewrite of `applyStaleRemoval`'s doc comment, each excepting the root scratch link.
- F4: card 1's `TestWireJunctions_ScratchLinkRefusedAfterClassify` and card 2's Reconcile rows added to it state that each row builds and arranges its hub before setting a seam.
- F5: the failed-exclude-write row's arrange asserts the `info` directory's exclude lock file exists before chmodding the directory `0o555`.
- F6: card 2's `TestReconcile_ScratchLink` gains the row with the weft-side `.lyx` replaced by a regular file, asserting the Detail names the root `.scratch` and `lyx fabric reconcile` and the root stays a real directory with its file.
  To make that row reach the real-directory branch, card 1's "target absent" now counts an `os.Lstat` that fails with not-a-directory as absent, since `os.IsNotExist` does not match `ENOTDIR`.
- F7: card 1's Intent places `SetScratchPathTrackedForTest` in `internal/fabricengine/export_test.go` beside `SetScratchBeforeLinkHookForTest`.
- F8: card 3's Uses drops `plan:internal/fabricengine#scratchLinkTarget`.

## Self-check

- fix-affected-set, LOW: `doc.go`'s `Unwire` paragraph still claims every on-disk junction is removed. Fixed as proposed: card 1's Docs paragraph names that paragraph's exception.
- fix-consistency, LOW: card 2's "the same three shapes" became ambiguous once card 1 gained a fourth row. Fixed as proposed: card 2 names the case-5 row and the two case-6 rows.
- fix-consistency, LOW: neither card's Docs paragraph named the weft-root check. Fixed as proposed: card 1's and card 2's Docs paragraphs name it.
- fix-claims, LOW: the line ranges cited for F3 were off. Fixed as proposed: `unwire.go:53-54`, `remove.go:242-247`, `reconcile.go:801-804`.
- fix-proposal, LOW: the F1 sentence claimed an absolute guarantee, but the stat-then-create check leaves a window open. Fixed as proposed: cards 1 and 2 now bound the claim to a teardown that happens before the check runs.

## Disputed

None.

## Deferred

None.
