```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I named the test row that pins it, or none.

**Behaviour → pinning test**
1. A package's files are grouped into one piece, in first-seen order, with names like `NNN-<pkg>.diff` and `root`: `TestWriteDiffPieces/OnePiecePerPackageInFirstSeenOrder`. Pinned.
2. A whitespace-only diff writes nothing and creates no directory: `TestWriteDiffPieces/EmptyDiffWritesNothing`. Pinned.
3. A package over the cap splits at file boundaries into numbered parts, named `-partN`: `TestWriteDiffPieces/PackageOverTheCapSplitsAtFiles`. Pinned.
4. A file over the cap splits at hunk starts, then at line ends, every byte kept and every piece under the cap: `TestWriteDiffPieces/FileOverTheCapSplitsAtHunksAndLines`. Pinned through the joined text, the cap check and the split check.
5. A single line over the cap is cut at the cap: `TestWriteDiffPieces/LineOverTheCapIsCut`. Pinned.
6. A failed `MkdirAll` or `WriteFile` in `writeDiffPieces` is returned, and `Engine.Review` returns it: **none**.
7. A section with no `diff --git` header, or a quoted path, goes to `rootPackage`: **none**. This is an edge case of behaviour 1.
8. `formatDiffPieces` labels a root piece "(the repository root)" and a split package "part N of M": **none**. `TestEngine_Review_StagesTheDiff` checks only the backticked path of single-part, non-root pieces.
9. `Engine.Review` writes the pieces under `diff/` and the explore step names each one; with no diff, nothing is written and the section is absent: `TestEngine_Review_StagesTheDiff/DiffIsStaged` and `/NoDiffStagesNothing`. Pinned.
10. An assistant reviewer's explore step names the same pieces (claimed in `doc.go:514` and `summary.md`): **none**. `assist_test.go` has no row with `Profile.Diff` set.
11. The `diff_pieces` marker is optional, renders empty cleanly, and sits after the target and before the fasit: the `TestTemplate_OptionalDirectives` `diff_pieces` row and `TestTemplate_FillsWithAllMarkers`. Pinned.
12. The `burler-diff-pieces` stencil is registered with its `piece_list` marker: the `TestStencils_TopLevelMarkers` row. Pinned.
13. `readDiff`: a reader's text reaches the spawned profile, a failed read spawns with no diff, and no reader gives no diff: the three `diffRows` in `TestBurlerReviewProducer_Call_ProfileAndOpts`. Pinned.
14. The diff is read once per spawning call and not on the hand-on or probe paths: **none**. No row counts reader calls or checks that the hand-on path skips the read.
15. `stage_diff: true` hands over the code subject's patch, and a subject that is not found gives no diff: `StageDiffGivesTheCodeSubjectsPatch/SubjectFound` and `/SubjectNotFound`. Pinned.
16. `subjectDiff` passes the subject's error on: **none** at the recipe level. Only the producer's generic failed-read row covers it.
17. `stage_diff` with a nil `Env.CodeSubject` is refused when the row is built: `StageDiffWithoutCodeSubjectIsRefused`. Pinned.
18. A row without `stage_diff` gives no diff: `NoStageDiffGivesNoDiff`. Pinned.
19. The tapestry `Code-Burler-Review` row sets `stage_diff`: likely pinned by `internal/loomrecipe/blockrecipes_test.go` against the updated `tapestry-rows-before-motif.yaml` fixture. I inferred this from the fixture edit and did not read the test.
20. The darn and sampler `Code-Burler-Review` rows set `stage_diff`: **none**.

**Findings**
- **MEDIUM**, `design` — `internal/burlerengine/diffpieces_test.go:452` (`TestEngine_Review_StagesTheDiff`). No test pins that an assistant reviewer's explore step names the diff pieces, though `internal/burlerengine/doc.go:514` and `_lym/landing/summary.md` claim it. Evidence: `internal/burlerengine/assist.go:83-90,186`; no assist test sets `Diff` (grep of `internal/burlerengine/*_test.go`).
- **LOW**, `design` — `internal/burlerengine/diffpieces_test.go:378-393`. Neither `formatDiffPieces` label is pinned: "(the repository root)" and "part N of M" (`diffpieces.go:309-314`). `TestEngine_Review_StagesTheDiff` checks only the backticked path.
- **LOW**, `design` — `internal/burlerengine/diffpieces.go:176-191` and `review.go:613-616`. The error path of a failed directory create or piece write is untested; it is a filesystem boundary failure.
- **LOW**, `design` — `internal/shedadapters/burlerreview_test.go:778-800`. The "read once per spawning call" contract is unpinned. No row counts reader calls across a died-then-retried attempt, or checks that the accepted and unjudged hand-on paths never call the reader (`burlerreview.go:376`).
- **LOW**, `design` — `internal/shedrecipe/entries_burlerhalves_test.go:900-929`. `subjectDiff`'s error pass-through has no row (`entries_burler.go:62-66`).
- **NIT**, `design` — `contracts/recipes/darn-recipe.yaml:197` and `sampler-recipe.yaml:159`. No test pins that these two rows set `stage_diff: true`; only tapestry's has a fixture.

Tool calls:
1. Grepped the tests for `stage_diff`, `diffPieces` and `Diff` outside the three changed test files, for consumers of the fixture, and for assist-test option rows. My context lacked whether any other test pins the assistant's pieces or the recipe rows.
