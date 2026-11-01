```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test row that would fail if it broke, or none.

**Behaviour → pinning test**
- `ParseData` accepts and rejects the same content as `Parse`: pinned by `TestParse` (`summary_test.go`). Every row with content runs both, so a divergence fails.
- `partOf` places darn, drift, code-review and landing roles: pinned by the new `TestPartOf` rows. Rows cover `darn`, `darn+sub`, `drift-baseline`, `code-burler-review`, `code-burler-fix+sub`, `code-bouncer-judge`, `describe`, `describe+sub`, `pr-reviewer`, `fixer` and `fix-reviewer`.
- Segment columns of "Cost per run" are only the segments some run holds, with a Landing column: pinned by `TestSegmentReport`'s `wantPerRun`. The Orch and Other column case is pinned by `TestCountOrch`.
- The recipe column and the blank recipe on the average row: pinned by the same `wantPerRun` and by `TestCountOrch`.
- Recipes section placement after "Cost per run": pinned by `TestSegmentReport`, whose expected text ends in `## Recipes\n`.
- `finishedRun` counts darn, skips other recipes, skips runs in flight and runs that stopped early: pinned by `TestRecentRuns`. The `darn` slug counts, `sampled` (sampler recipe), `bench` and `inflight` are skipped.
- A darn run with no Finalize entry is not counted: none for darn specifically. The tapestry `bench` row covers the shared Finalize check, so this is not a gap.
- `recentRuns` `since` bound: pinned by the `every run since a time` row, which counts `darn` only. The inclusive edge (`!touched.Before(since)`) is not pinned by any row.
- `recentRuns` with `last <= 0` unlimited: pinned by the same `since` row, since `last` is 0 there.
- Locator failure is still an error: pinned by `TestRecentRunsLocatorFailure`.
- `parseSince` rejects a non-date: pinned by the `TestRunRefuses` row `a since that is no date`.
- `parseSince` of a valid date, and `-since` overriding `-last` in `run`: none (F1).
- New header and error wording ("finished tapestry and darn runs", "Skipped N runs that are not finished…"): pinned by `TestRunReadsArchivedShedStatus`'s `wantHeader` and `wantErr`.
- The `active since <date>` header: none (F1).
- `findLanding`: `TestFindLanding` pins the newest commit within the span, the one-minute slack past the end, no titled commit, no description, and a description that does not parse.
  - The slack before the span start: none.
  - A failing `CommitsWithSubject` or `ChangedLines`: none.
  - `FileAtRevision` failing with an error other than path-absent: none.
- `ChangedLines` and `numstatLines` through a real git repository: pinned by `TestRunReadsArchivedShedStatus`, where the Recipes row for the full run carries 3 changed lines.
  - The assertion is only the substrings `"| tapestry | 1 | "` and `" | 1 | 3 | "`, so it checks neither the column nor the row it sits in.
  - Binary files (`-` counts), deleted lines, and a landed commit with no first parent: none (F3).
- `writeRecipes` grouping, sorted names, cost per run, span per run, lines per landed run, cost per changed line over landed runs only, the `unknown` recipe, and the "No landed commit" list (nil landing and a missing reason): pinned by `TestWriteRecipes`.
- `writeRecipes` with landed runs whose lines total 0: none (F2). The `row.lines > 0` guard has no row.
- `Cost.precise` unpriced branches: none (F2).
- The `-code` flag, and `Calibrate` now receiving the `-code` repository instead of the current directory: none. The integration test passes `-code` but does not run `-calibrate`.
- `tokencount.sh` and `tokencount.cmd` usage text: not testable and exempt.

**Findings**

- **F1, MEDIUM, class `scope`.** `-since` has no behavioural test at run level. `tools/tokencount/main.go:283-296` (the `limit = 0` override and the `active since` header) is not exercised by `time_integration_test.go`, which only passes `-last 1`. A valid-date `parseSince` has no row either (`main.go:212-221`). `TestRecentRuns` pins the `since` argument of `recentRuns`, but nothing pins that `run` passes it and drops `-last`. Fix: add a `-since` run to `TestRunReadsArchivedShedStatus` asserting the header, or a `TestParseSince` row.

- **F2, LOW, class `scope`.** The zero-line division guard in `writeRecipes` (`recipes.go:563-568`: `landed > 0` but `lines == 0`, so lines per run is `0` and cost per line is `n/a`) and `Cost.precise`'s unpriced branches (`cost.go:63-73`) have no test row. `TestWriteRecipes` only covers a recipe with no landed runs. Fix: add a darn run landing 0 lines and an unpriced-cost run to `TestWriteRecipes`.

- **F3, LOW, class `scope`.** `ChangedLines` (`landed.go:256-278`) diffs `sha^1..sha`. No test covers a landed commit with no parent, which would make `git diff` fail and, via `findLanding`'s error return and `main.go:331-334`, abort the whole report. A binary file and a deletion in the numstat parse are also unpinned. Fix: add a root-commit or binary-file case to the real-git test, or a table test through the public surface.

- **F4, NIT, class `scope`.** `TestFindLanding` has no row for a commit inside the one-minute slack before the span start, and no row for the error paths of `CommitsWithSubject` or `ChangedLines` (`landed.go:296-318`). Fix: add rows with those fakes returning errors.

No redundant or duplicate tests found. The new `TestRecentRunsLocatorFailure` is the old locator assertion split out, and `TestWriteRecipes` and the integration Recipes assertion check different layers.

Tool calls: none; the manifest context held every diff and definition this lens needed.
