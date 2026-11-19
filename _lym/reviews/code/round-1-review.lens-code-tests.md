```yaml
lens: code-tests
outcome: reported
```

Restating my task: under the code-tests lens, find for each behaviour the diff adds or changes the test row that fails if that behaviour breaks.

**Per behaviour, the test that pins it**

1. `LoadConfig` keeps a set `report_dir` in clean slash form. Pinned by `TestLoadConfig` row "a report_dir loads in clean form" (`internal/darnengine/config_test.go:107`).
2. `LoadConfig` refuses a `report_dir` that escapes the worktree, and the message names the key, the value and the way forward. Pinned by `TestLoadConfig` row "a report_dir outside the worktree is refused" (`config_test.go:112`). An absolute path takes the same `filepath.IsLocal` branch, so it needs no row of its own.
3. `report_dir` is classified as `CategoryNone` and so stays open to the local layer. Pinned by `TestModules_EveryTemplateKeyClassified` (`internal/configreg/configreg_test.go:151`).
4. An empty `ReportDir` renders no passage and keeps the prompt's bytes unchanged. Pinned by `TestDarnSpec`'s first row, whose `wantNotInPrompt` includes `reportHeading` (`spec_test.go:113`). Every other existing row also passes an empty `report_directive`, which exercises the `Optional` entry.
5. A set `ReportDir` renders the passage right after the Step 4 list, with `report_dir` filled and `<dir>/<slug>.md` as the default path. Pinned by `TestDarnSpec` row "a report directory has the writer commit the report there…" (`spec_test.go:215`).
6. The `docs` profile sets `report_dir: docs/research`. Pinned by `TestTemplateDocsProfile` (`internal/runprofile/runprofile_test.go:232`).
7. `landedDiffEmpty` covers four cases: an empty path list gives true, a non-empty list gives false, a failed read gives false, and no merged head gives false. Pinned by `TestFinalize_RecordsWhetherTheLandedDiffIsEmpty`, rows NoPathChanged, APathChanged, ThePathReadFails and NoMergedHeadHeld (`finalize_test.go:135-138`). The `ChangedPaths == nil` guard has no row. It is a defensive branch for a caller-controlled seam, so I file nothing on it.
8. `recordLanding` hands the flag to `RecordLanding` (`finalize.go:741`). Pinned by the same test, which asserts the recorded flags. The mutation survivor at line 741 negates `err != nil`, and that only changes which branch logs, so no test can reasonably kill it. No finding.
9. `WriteLandedCommit` adds the `empty-diff` second line and `ReadLandedCommit` round-trips it. A missing file reads as the zero value, and a commit-only file with a trailing newline reads as not empty. Pinned by `TestLandedCommit` (`internal/shedrun/landed_test.go:22-40`). The survivor at `landed.go:43` changes the file mode constant, which no behaviour depends on. No finding.
10. `DobbyRuns` carries the flag into `RunView.EmptyLanding`. Pinned by `TestDobbyRuns_ReadsTheLandOutcomeTheLandingCommitAndTheEndedEntryAndPair`, row "landed" with `emptyLanding: true` (`internal/beamcli/dobbypairanchor_test.go:283,329`).
11. The `RecordLanding` seam in `landingDeps` writes the flag where dobby reads it. Pinned by `TestLandingDeps_RecordLandingWritesWhereTheDobbyReads` (`internal/loomcli/landingdeps_integration_test.go:340-337`).
12. `landNoticeLine` names an empty landing. Pinned by `TestLandNotices` row "AllEmptyLandingIsFlagged" (`internal/dobbyengine/landnotice_test.go:86`).
13. An empty landing's line ends with a beam member's follow-up. No test pins this; see F1.

**Findings**

- **F1 — LOW, class design.** Location: `internal/dobbyengine/landnotice.go:317-318`. The empty-landing branch adds a new format string that ends in `followUp`, and `_lym/landing/summary.md` claims "A beam member's follow-up still ends the line". No row combines `landedEmpty` with a beam member: the member rows at `landnotice_test.go:51-52` use `landedLine` only. A mutant that drops `%s followUp` from the empty branch passes every test. Fix: add a member row with `landedEmpty`, whose want is `emptyLandingLine + memberFollowUp`. Evidence: `internal/dobbyengine/landnotice_test.go:47-52,86`.
- **F2 — LOW, class consistency.** Location: `internal/darnengine/spec_test.go:184-188` and `:220`. The new row asserts static stencil prose: `reportHeading` ("## The report this run commits"), `reportAfterStep4`, which embeds the make-change fragment's last line "Never push: the landing steps that follow push the branch.", and the phrases "under `…/`" and "at `…`". A wording edit to either fragment then breaks this test. The rubric's test-economy bullet names "a new assertion on the static prose of a stencil". Fix: pin the filled values (`docs/research/`, `docs/research/fix-the-thing.md`) and their position after the Step 4 list, without quoting the fragments' wording. Evidence: `spec_test.go:184-188,220`.

Tool calls: none.
