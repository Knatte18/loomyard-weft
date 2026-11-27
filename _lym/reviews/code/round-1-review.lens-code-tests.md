```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds, the test row that would fail if it broke.

**Per behaviour, the test that pins it**

1. **No recorded outcome, but branch commits and scratch files exist → the "interrupted" note with both lists.** Pinned by the `TestDarnWrite_Call` row "an interrupted spawn that recorded no outcome is named by its commits and scratch files" (`internal/loomshed/darnwrite_test.go`). The same row pins that the producer follows a linked scratch directory (`writeLinkedScratch`), that it lists nested paths with slashes, and that it sorts them lexically (`measure.md` before `runs/second.txt`).
2. **Scratch files alone name an interrupted spawn, and the list is cut at 20 with a count.** Pinned by the `TestRenderPriorWork` row "scratch files alone name an interrupted spawn and a long list is cut with a count".
3. **After a recorded outcome, the commit list follows the findings line.** Pinned by the `TestRenderPriorWork` row "an earlier entry lists the branch's commits after its findings".
4. **After a recorded outcome, the scratch-file list follows.** No row. The only scratch-file rows have no recorded outcome. It shares code with item 3, but nothing pins its order after the outcome lines.
5. **No outcome, no commits and no scratch files → empty prior work, and an absent scratch directory holds no file.** Pinned implicitly. Every `TestDarnWrite_Call` row without `scratchFiles` points `ScratchDir` at a missing path and expects the empty told text. `TestRenderPriorWork` also has "nothing to name".
6. **A `BranchCommits` error halts the call.** Pinned by the `TestDarnWrite_Call` row "branch commits read failure is an error".
7. **A `listScratchFiles` error halts the call (unreadable entry, stat error other than not-exist).** No row.
8. **A dangling `.scratch` link reads as no files.** No row.
9. **An unwired `BranchCommits` or empty `ScratchDir` is refused by `checkSeams`.** Pinned by the rows "unwired BranchCommits seam is named" and "unwired ScratchDir is named".
10. **The registry refuses a nil `Darn.BranchCommits` or a relative `Darn.ScratchDir`.** Pinned by `TestDarnWriteEntry_ConstructionFailures` rows `NilBranchCommits` and `RelativeScratchDir`.
11. **`loomcli.darnDeps` wires `BranchCommits` as `CommitLog(taskMergeBase, HEAD)`.** None. `TestWire_DarnRecipe`'s `darnSeams` list (`internal/loomcli/wiring_darn_test.go:81`) still lacks `Darn.BranchCommits`. No test calls `darnDeps(...).BranchCommits()` against a git fixture. The only `darnDeps` test, `TestDarnDeps_LatestOutcomeNamesTheRowsFindingsFile` (line 892), covers `LatestOutcome` alone.
12. **`darnDeps` sets `ScratchDir` to `fabricengine.ScratchLinkPath(l)`, and `ScratchLinkPath` returns `&lt;anchor&gt;/.scratch`.** None. No test anywhere references `ScratchLinkPath`, and no test asserts `Darn.ScratchDir`.
13. **A non-darn wire leaves `Darn.BranchCommits` nil.** Pinned by `TestWire_EverySeamFilled` in loomcli and in battencli, via the `intentionallyNil` entries.
14. **Expected-value independence.** The `TestDarnWrite_Call` row that used `renderPriorWork(...)` as its expected value now uses a literal, so it is no longer tautological. No new tautology.

**Findings**

- **MEDIUM · design · `internal/loomcli/darnwiring.go:200-211`.** No test pins the production wiring of `BranchCommits` (merge base to `HEAD` through `CommitLog`) or of `ScratchDir` (`fabricengine.ScratchLinkPath`). Each of these would pass every test if broken: a wrong base, a wrong range direction, a scratch path off the link, or a dropped seam. Evidence: `wiring_darn_test.go:81` (`darnSeams` lacks `Darn.BranchCommits`) and `wiring_darn_test.go:892`, the only `darnDeps` test, which calls `LatestOutcome` only. A grep for `ScratchLinkPath`, `.BranchCommits` and `Darn.ScratchDir` across `*_test.go` finds only the `intentionallyNil` entries and the shedrecipe construction rows.
- **LOW · design · `internal/fabricengine/scratchlink.go:67-70`.** `ScratchLinkPath` is a new exported function with no test of its result. It also has no assertion that it matches `scratchLinkPath(l, slug)` at a subpath anchor. The same grep found no test reference.
- **LOW · design · `internal/loomshed/darnwrite.go:320-337`.** `listScratchFiles` has no test of its failure path: an unreadable or vanishing entry makes `WalkDir` return an error, which halts the row. It also has no test of a dangling link reading as empty. Both behaviours are reachable, and no row in `TestDarnWrite_Call` covers them (`darnwrite_test.go`, only the linked-directory happy-path row).
- **NIT · design · `internal/loomshed/darnwrite_test.go` (`TestRenderPriorWork`).** No row pins the scratch-file list after a recorded outcome, which the summary and the `renderPriorWork` doc say follows the existing note. Only the commit list has a recorded-outcome row.

**Tool calls**

1. Grep of `*_test.go` for `ScratchLinkPath`, `BranchCommits` and `ScratchDir`: the manifest did not show whether any test covers the new wiring. The output was too large, so I narrowed it in call 2.
2. Narrowed grep, plus the body of `TestWire_EverySeamFilled`: to see whether it asserts any darn seam is filled.
3. Grep of `wiring_darn_test.go` for its test names and seam lists: to find a darn-seed seam test.
4. Read of `wiring_darn_test.go:77-126`: to see whether `TestWire_DarnRecipe`'s `darnSeams` lists `Darn.BranchCommits`.
