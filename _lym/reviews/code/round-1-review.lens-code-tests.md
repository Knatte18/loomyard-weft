```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Per behaviour, the test that pins it:**
1. A new untracked file fails a passing tier with exit 1 and outcome failed: `TestRunTierStrayFiles/ANewFileFailsAPassingTier`.
2. A failing tier keeps its own exit code, and the line names the files sorted: `TestRunTierStrayFiles/AFailingTierKeepsItsExitCode`.
3. A file that was untracked before the run is not named: `TestRunTierStrayFiles/AFileUntrackedBeforeTheRunIsNotNamed`.
4. An ignored file is not named (`--exclude-standard`): `TestRunTierStrayFiles/AnIgnoredFileIsNotNamed`.
5. A file the run creates and then removes is not named: `TestRunTierStrayFiles/AFileTheRunRemovesIsNotNamed`.
6. The cap of `strayFilesListed` names, with the rest counted as "and N more": `TestRunTierStrayFiles/MoreFilesThanTheLineNamesAreCounted`.
7. A worktree git cannot list is not checked (first listing fails): `TestRunTierStrayFiles/ADirectoryOutsideEveryRepositoryIsNotChecked`.
8. The line goes to stderr, the causes and the log: asserted in every row of `TestRunTierStrayFiles` that expects a line.
9. A run with no invocation takes no listing (`tierrun.go:74`): no test. The behaviour is not observable, since a run with no invocation creates no file. No finding.
10. A result with a refusal skips the check (`tierrun.go:83`): none.
11. A `--copies` run is listed once around all its copies, and the stray result merges into the copies result (`tierrun.go:78–86`, `copies.go:94`): none. The tests in `copies_test.go` run in a plain `dir` with no git repository (`copies_test.go:91`, `copies_test.go:244`), so the check is skipped there.
12. A git failure on the listing after the run leaves the result unchanged (`strayfiles.go:41–43`): none. It is hard to reach without a fault seam. No finding.
13. `runTierCommands` keeps the body that used to be `runTier`: the existing `TestRunTier*` tests and `TestRunTierCopies*` (see the reach report).

**Findings:**

- **MEDIUM, design.** `internal/gatecli/tierrun.go:78-86` together with `internal/gatecli/copies.go:94`. A `--copies` run that leaves a stray file has no test.
  The copies path is the one composition with its own result shape: `mergeCopies` carries `copies`, and `addFailedRuns` reads each copy's own exit code and causes.
  The diff's doc claim "A `--copies` run is listed once around all its copies" (doc.go `# Stray files`) is not pinned anywhere.
  Evidence: `TestRunTierStrayFiles` never sets `copiesRun` (strayfiles_integration_test.go:66), and the copies tests at copies_test.go:91 and :244 use a root with no `git init`.
  Fix: add a row to `TestRunTierStrayFiles` with `copiesRun: 2`. It should assert that the merged result fails with the line in `causes` once, and that the line reaches the failure report a red copy run produces.
- **LOW, design.** `internal/gatecli/tierrun.go:83`. The refusal skip has no row.
  Mutation survivor `tierrun.go:83` names it.
  A run whose second invocation is refused after the first invocation wrote a file would show it. One example is a tier with two invocations where the second's dir does not exist, which makes "could not start the shell" a refusal at tierrun.go:479-481.
  Evidence: no row of `TestRunTierStrayFiles` produces a refusal. Each row passes `invocations: []invocation{{dir: root}}` (strayfiles_integration_test.go:67).
- **NIT, design.** On the mutation report: the survivors at `strayfiles.go:21`, `tierrun.go:74` (negated), `:75` and `:83` (the `!listed` half) would each make the rows of `TestRunTierStrayFiles` that expect a line fail.
  They most likely survived because that tier1 test was outside the mutation run; the redundancy report marks it "not judged, out of process".
  None of them is a missing check, so this is no finding beyond the two above.
- **Tautology check: none.** The expected values come from literal file names and the literal prefix "gate test: tier1 left new untracked files in the worktree: ". The one derived value is `many[:strayFilesListed]`, a slice of literal names cut at the constant whose cap is the contract. The tests do not recompute the line through `strayFilesLine`.

**Tool calls:**
1. Grepped the `TestRunTierCopies` body for root and git setup. It returned nothing, because the pattern missed. My context did not show whether the copies tests run in a git repository.
2. Grepped `copies_test.go` for `copiesRun`, `gitkit` and `root:` to settle the same question.
