```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test row that would fail if it broke.

**Behaviours and the tests that pin them**

1. A changed non-Go file that a production file at the base embeds by name is restored to its base text. Pinned by `TestBaseOverlay_ReplacesTheFilesOfTheModuleTheBasesBuildReads/NonGoFileAProductionFileEmbedsMapsToACopyOfTheBaseText` (`internal/gatecli/redfirst_test.go`). With the real go command, `TestGateRedFirst/ATestOfAnEmbeddedFileTheDiffChangesRunsOnTheBasesText` (`redfirst_integration_test.go:125-135`) pins it too: without the overlay, `Text` reads "fixed", so the run comes out green and the exit-0 assertion fails.
2. A file under an embedded directory, with the `all:` prefix stripped and the ancestor walk in `embedPatternMatches`, is restored. Pinned by `…/NonGoFileUnderAnEmbeddedDirectoryMapsToACopy` (`sub/a/embedded/deep/data.json` against `all:embedded`).
3. A double-quoted glob is restored, and so is a directive with several patterns. Pinned by `…/NonGoFileAQuotedGlobEmbedsMapsToACopy`: the `a.go` directive line holds three fields.
4. A file that a package above embeds is restored, and the walk reaches past a directory whose Go file embeds nothing. Pinned by `…/NonGoFileAPackageAboveEmbedsMapsToACopy` (`inner/*.txt`, with `sub/a/inner/i.go` present).
5. An embedded file that the base lacks maps to "". Pinned by `…/EmbeddedNonGoFileTheBaseLacksMapsToNoCopy`.
6. An embed directive in a `_test.go` file does not count. Pinned by `…/NonGoFileOnlyATestFileEmbedsIsLeftAlone`: removing the `_test.go` skip in `embeddedAtBase` would replace `fixture.json`.
7. A non-Go file that nothing embeds is left alone. Pinned by `…/NonGoFileNoGoFileEmbedsIsLeftAlone`, `…/NonGoFileWithNoGoFileAtOrAboveItIsLeftAlone`, and `TestGateRedFirst/ANonGoFileNoProductionFileEmbedsLeavesTheVerdictToTheTests`. The integration row expects the green exit, where the old code gave exit 5.
8. `testdata` and test files are still left alone after the `_test.go` check moved up front. Pinned by `…/FileUnderTestdataIsLeftAlone`, `…/NonGoFileUnderTestdataIsLeftAlone` and `…/TestFileIsLeftAsTheWorkingTreeHoldsIt`.
9. The "cannot put back" refusal is gone from `redFirstCall.run`. Pinned by `TestGateRedFirst/ANonGoFileNoProductionFileEmbedsLeavesTheVerdictToTheTests`, run in-process through `clihelp.ExecuteIn`.
10. The loomshed exit-5 failure text now names only the scoper. Pinned by `TestRedFirstGate_VerdictPerLabelRecordAndExitCode`, in the row "a check that exits 5 …" (`internal/loomshed/redfirstgate_test.go:106-112`). This pins a runtime output string, not static prose. The assertion was narrowed, not loosened.
11. Backtick-quoted embed patterns are stripped (the `` "`" `` in `strings.Trim(field, "\"`")`, `redfirst.go:438`). No test pins this.
12. A read error from the base tree in `embeddedAtBase` (`goFiles` or `file`) propagates out of `baseOverlay` and `run` as a refusal with "re-run" as the way forward. No test pins this: `memoryTree` never returns an error.
13. The help text, the doc comments and the stencil are static prose, so this lens exempts them.

No base assertion was widened. Removing the `unrestorable` checks and changing the expectation of the old integration row follow from the behaviour change the Goal asks for. Expected values come from literals (`baseText`, `"fixed\n"`, the `red_tests` label), so no test is tautological.

**Findings**

- **LOW, design.** `internal/gatecli/redfirst.go:438` (`embedPatterns`). No test covers stripping a backtick-quoted pattern. No row of `TestBaseOverlay_ReplacesTheFilesOfTheModuleTheBasesBuildReads` holds a backtick pattern, so a mutant that drops `` ` `` from the trim set survives. Evidence: the `baseFiles` map in `redfirst_test.go` has only bare and double-quoted patterns. Fix: add a backtick-quoted pattern to `sub/a/a.go`'s directive and a row that expects the file to map to a copy.
- **LOW, design.** `internal/gatecli/redfirst.go:405-417` (`embeddedAtBase` error returns) and `redfirst.go:371-374` (their propagation out of `baseOverlay`). No test covers a base-tree read error, which comes from git through `commitTree`: a trust boundary. Evidence: `memoryTree.file` and `memoryTree.goFiles` (`redfirst_test.go:26-40`) always return a nil error. Fix: add a row to the `baseOverlay` table with a tree whose `goFiles` or `file` fails, and assert that `baseOverlay` returns an error and no replacements.

Tool calls: none. The inherited context held every file this lens needed.
