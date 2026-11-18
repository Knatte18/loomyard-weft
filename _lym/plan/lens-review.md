# Lens review

Every lens fork returned a report: coverage, row-validity, forced-edits, guard-reach and way-forward.
Duplicates are merged on the first line that names them.

- coverage 1 (BLOCKING), guard-reach 2 (BLOCKING): fixed differently — the proposal was to have cards 6 and 14 extend the guard's pin list; the clean fix keeps the pins to today's readers. Card 2 adds `testreach.ReadPackageMap`, which cards 6 and 14 use for their single-package reads, and card 4's failure text names it.
- coverage 2 (MEDIUM): fixed as proposed — card 6 adds rows where a stale and an absent anchor map are refreshed, the absent one's linked packages coming from the stand-in's dependency list, and each map is written.
- coverage 3 (MEDIUM): fixed as proposed — card 6 adds a row for a task worktree with no parent branch.
- coverage 4 (MEDIUM), guard-reach 4 (MEDIUM): fixed as proposed — card 9 adds a `TestWrapRowGate` subtest that composes `WrapRowGate` over `WithReachTurn` with one shared `RerunMark`: no fixer for a reach turn, and the note is returned after a fixer's re-run.
- coverage 5 (MEDIUM): fixed as proposed — card 8 adds a row whose re-run after the fixer returns a `WriterOnly` result to the writer with the fixer's reply.
- coverage 6 (MEDIUM): fixed as proposed — card 6 adds a step row where a package holding a tier2 test file gets the caveat.
- coverage 7 (MEDIUM): fixed as proposed — card 6's `ReachReportMatches` rows add a commit changing a changed function's text and one adding a changed function.
- coverage 8 (MEDIUM): fixed differently — a new card-fixer test was not added, because `internal/gatecli#TestRunWithFixer` (`cardfixer_test.go:301`) already pins that the fixer report is the run's stderr verbatim. Instead, card 11 edits `TestGateTest_Scenario` to pin the triage line in that stderr's failure report, which also reaches `printReport`'s wiring, and cites both tests.
- coverage 9 (MEDIUM): fixed as proposed — card 11's `newTriageLookup` takes the merge-base read as a function, and `TestTriageLookup_Lines` has a row for each of the two no-change-set causes.
- coverage 10 (MEDIUM): fixed as proposed — card 10 adds a row where a worktree with no parent branch walks its current branch.
- coverage 11 (MEDIUM), way-forward 4 (MEDIUM): fixed as proposed — card 10 defines the output for a test no map holds (says so, exit 0, no commit listed) and a refusal for a `--test` with no dot, each with a row.
- coverage 12 (MEDIUM): fixed as proposed — card 14 adds a row where a stale package with committed files is refreshed and then judged.
- coverage 13 (MEDIUM): fixed as proposed — card 2 adds rows for a `BuildFailure`-only map, an out-of-table map and an unparsable index.
- coverage 14 (LOW): fixed as proposed — card 14 adds a memory-cap unjudged row.
- coverage 15 (LOW): fixed as proposed — card 14's kill row asserts the block's second covering test does not run.
- coverage 16 (LOW): fixed as proposed — card 14 adds a cap-of-two row over two changed functions, one mutant each.
- coverage 17 (LOW): fixed as proposed — card 3 adds assertions for the HEAD-moved row (no index) and a failed index write (output and exit unchanged).
- coverage 18 (LOW): fixed as proposed — card 9 adds rows for a judged-nothing report, a failed listed-file record and the count reset after a pass.
- coverage 19 (LOW): fixed as proposed — card 6's note rows assert each map's age; card 11's rows assert the map's commit.
- coverage 20 (LOW): fixed as proposed — card 1 adds a row for a file outside the module, under `testdata` and under a `.`/`_` directory.
- coverage 21 (LOW): fixed as proposed — card 12 adds a rerun row.
- coverage 22 (LOW): fixed differently — a guard over the HEAD-match clause was not added, because no scan can tell a matching read from a non-matching one; the `TestRenderReachReportNote` and `TestRenderRedundancyReportNote` rows pin that clause. Instead, card 6 extends the guard to pin `testreach.ChangedFuncs` to gatecli's one `changedFuncsAt` file, which covers the changed-functions clause.
- row-validity 1 (MEDIUM): fixed as proposed — `reachStep` gains `minuteUnit`, and the budget row drives a hand-built step as `stepAt` does.
- row-validity 2 (MEDIUM): fixed as proposed — card 8 moves `scriptedGate` to Edit; it gains a `RerunMark` and a per-call record.
- row-validity 3 (MEDIUM): fixed as proposed — card 13 adds `lineMutantFixture`, a package of its own holding a logger import and call, so `planFixture` and `TestPlanMutants`' constants stay.
- row-validity 4 (LOW): fixed as proposed — card 2's stale-index row changes a reach pair and moves the modification time with `os.Chtimes`.
- row-validity 5 (LOW): fixed as proposed — card 9's batch test puts `VerifyDir` in a temp directory outside the git worktree.
- row-validity 6 (LOW): fixed as proposed — card 12 names the `triagedPackageLog` fixture.
- row-validity 7 (LOW): fixed as proposed — card 11 drops the at-cap `TestFailedTests` row.
- row-validity 8 (LOW): fixed as proposed — card 14 names the real `go` toolchain (`defaultGoBinary`), as `TestGateRedundant`'s mutation rows use.
- forced-edits 1 (MEDIUM): fixed as proposed — card 11 lists `TestGateTest_Scenario` and states its new assertion.
- forced-edits 2 (MEDIUM): fixed as proposed — card 9 names `WeaverDeps.Geom.ReachPending` in `nilLegal`.
- forced-edits 3 (LOW): fixed differently — the `testmap.go:2` file comment and `doc.go:5` were not changed, since both still hold: `lym gate test --map` itself runs only on command. Card 6 moves `internal/gatecli/doc.go` to Edit and rewrites its line 1030 sentence and the overview's lines 842 and 974.
- forced-edits 4 (LOW): fixed as proposed — card 6 (the reach refresh) and card 14 (`lym gate mutate`) each update the `template.yaml` comments, the `doc.go` memory-guard sentence and the PATTERN-test-speed run list, which drops its count "Three".
- forced-edits 5 (LOW): fixed as proposed — card 13 states the change: `fileSites` keeps its signature, and its call sites gain a callee import-path field.
- guard-reach 1 (BLOCKING): fixed differently — the no-parent note was not simply moved after the prime check, which would leave the prime with no base. Instead, the prime diffs the working tree from HEAD, as `touchedFiles` does. A prime report carries `Prime` and never matches a review's HEAD (card 6, Shared Decisions); card 11's triage does the same. Card 14's prime row passes `--base`.
- guard-reach 3 (MEDIUM): fixed as proposed — card 9 moves the count once per arrival: a failure or a turn inside `RerunMark.On()` adds nothing, and a row pins it.
- guard-reach 5 (LOW): fixed as proposed — card 11 drops the per-test lookup-failure branch. D7's failed lookup is `open` failing in `newTriageLookup`, which builds no lookup, and a row reaches it.
- way-forward 1 (MEDIUM): fixed as proposed — `ReachPendingOf` answers false with a Warn for an unreadable or unmatchable report. Only a failed listed-file record stays a gate error.
- way-forward 2 (MEDIUM): fixed as proposed — card 10 resolves `--since` through `gitrepo.Repo.ResolveSHA`, adds a refusal for a rev naming no commit, and adds a short-SHA row.
- way-forward 3 (BLOCKING): fixed as proposed — `FirstParentCommits` resolves the branch tip to a SHA before the log read, and card 10 adds an unknown-`--branch` refusal with a row.
- way-forward 5 (MEDIUM): fixed as proposed — cards 10 and 14 print a note naming a non-`go` scoper with exit 0, each with a row.
- way-forward 6 (LOW): fixed as proposed — card 2's `Open` passes over an unparsable map with a Warn naming the package and `lym gate test --map`.
- way-forward 7 (LOW): fixed as proposed — card 6's unrefreshed lines and card 14's not-judged reasons each name their way forward.
- way-forward 8 (LOW): fixed as proposed — card 4's guard failure names its fix.
