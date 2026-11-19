```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set for agreement with its surroundings, and for twins left unchanged.

Findings:

- **LOW**: `internal/loomcli/wiring_commitstatus_integration_test.go:396`
  - The comment says "A records cherry-pick in a two-repository hub is fabric's own replay."
  - The fixed docs say a cherry-pick does not count, because fabric's replay cherry-picks under the write lock. The two now read differently.
  - The test comment is not wrong, but it was not aligned with the new wording.
  - The test is not in the changed set, so this is a stale sibling a reader could notice.

- **LOW**: `internal/fabricengine/doc.go:2487-2494` and `onerepository.go:29-36`
  - The mismatch is in `layoutAnswers`'s doc, which says "Fabric's own marker writes in this process clear the map".
  - Fabric marker writes that do not call `forgetLayouts` (the anchor move commit, `Add`'s records commit, the integrate re-root, `commitEmptySnapshot`) are covered only by the `forgetLayouts` and `doc.go` wording, "adds or retires".
  - So this reads as broader than the code. The `doc.go` clause "a rewrite of a marker that stays leaves them" narrows it.
  - It is consistent, but only because of the `doc.go` clause. Only `onerepository.go:30` says "marker writes" without that bound.

Checked and consistent:
- The `forgetLayouts` doc matches all three call sites: `anchormarker.go:85`, `anchormarker.go:211` and `clone_onerepository.go:293`. Each adds a marker or retires one.
- The `MergeStateActive` doc, `doc.go:1352` and `recordsduringmerge.go:144-145` now agree.
- The new test file's comment on non-parallel use matches the process-wide clear.
- `layoutClears` is incremented before `Clear`, which matches the doc comment.

Further calls:
1. Grepped the `.go` and `.md` files outside `_lym` and `.lym` for "revert or rebase", "own weft replay", "own replay", "waits out on the write lock", "kept for the process", "drops every kept" and "fixed once its marker", to find twins of the changed text. The context held only the files the review named.
2. Read `clone_onerepository.go:285-296` to check the third `forgetLayouts` call site, which the review did not name.
