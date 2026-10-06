# Decision record — test-suite-prune-core

## Goal

Shrink the test suites of `internal/fabricengine`, `internal/websterengine`, `internal/reedengine`, `internal/shuttleengine`, `internal/loomcli` and `internal/shedadapters` without losing a pinned behavior.
Delete tests the redundancy tool shows to be redundant once each is checked for an assertion-only contract, fold near-identical tests into table-driven tests, and move integration tests that need no git, tmux or process to the untagged tier.
Make the tool able to judge these packages first: in its committed report most of their tests sit in the "no coverage" list (spawns or unclassifiable call), outside its judgment.

## Scope

In:

- The test files of the six packages: delete, fold, retag, and move to `internal/testkit` fakes.
- `cmd/testtiming`'s redundancy mode: a narrower exclusion rule, a keep directive with a reason, and a section-splicing write of the report.
- `internal/testkit`: new or extended fakes the moved tests need.
  Shared test helpers belong to this task, not to `test-suite-prune-rest`.
- `docs/benchmarks/test-redundancy.md`: the six packages' sections, regenerated.
- `pattern/PATTERN-test-economy.md` (background) and `docs/benchmarks/running-tests.md`: the new tool semantics and the keep directive.
- A production seam in one of the six packages, only where a moved test needs a fake in place of git, tmux or a process.

Out:

- Test files of any package outside the six; they are `test-suite-prune-rest`'s.
- The report sections of packages outside the six.
- The after-state in `docs/benchmarks/test-suite-timing.md`, the per-package budget guard, gate tier selection and cgo link cost: `test-tiers` owns them.
- Speed work beyond what deletion, folding and retagging give.

## Decisions

### D1 — Exclusion means "module code ran out of process"

Decision: the redundancy tool excludes a test from judgment only when it may run this module's code in another process: it references `internal/testkit/lyxbin`, re-executes its own binary (`os.Executable`, `os.Args[0]`), invokes the `go` toolchain, or lives in a `tmux`-tier file (reed panes run `lyx`).
Spawning git or another external binary no longer excludes a test.
A test in an untagged file is never excluded as spawning or unresolved, because `PATTERN-test-speed` forbids an untagged test to spawn.
A test in a tagged file with a call the scan cannot resolve stays excluded.

Rationale: coverage misses only code that runs in a child process, and git's own execution is not module code, so a test that runs git keeps every block it reaches in-process.
The current rule leaves most tests of these packages outside the report, which makes the tool useless for this task.
Bound: the change only adds candidates to the report.
Every candidate is still checked by hand for assertion-only contracts before deletion (D4), so a wrong classification costs review time, never a silent deletion.
The scan does not detect a hook, alias or helper a test installs so that git runs module code; D4's hand check therefore asks whether the candidate makes git run module code, and such a candidate is kept under D2.
An untagged test that spawns anyway breaks `PATTERN-test-speed`, and its under-reported coverage can only turn it into a candidate that D4's check sees.

### D2 — A kept candidate carries `//testtiming:keep <reason>`

Decision: a candidate the prune keeps carries a `//testtiming:keep <reason>` directive comment directly above its `func Test…` line.
The tool lists such a test under a separate "Kept" list in its package section, with the reason, instead of as a candidate.
An empty reason, including one of only whitespace, is an error: the tool exits non-zero naming the file and test.
The reason names what the test pins that its covering tests do not (an error message, an output shape, an ordering).

Rationale: "no redundant candidate left without a stated reason" is checkable only if the reason is where the tool reads it, and the test source is the one durable place a rerun sees.
Bound: the directive changes report output only, never which tests run, and the reason is printed beside the test, so a keep is always visible.
A directive counts only as the comment line directly above its test function.
A second directive on one test, or a directive whose next line does not start a top-level `func Test…`, is an error naming file and line, like the empty reason.
A kept test stays in the covering pool and in the removable computation.
A directive on a test that is not a candidate is listed under "Kept" with its reason.

### D3 — A narrowed run rewrites only its own sections

Decision: when `-redundancy` runs with a `-pkg` narrower than the module and the `-out` file exists, the tool replaces only the sections of the packages it measured and leaves every other section and the header in place.
A package section that no longer has tests is removed.

Rationale: this task and `test-suite-prune-rest` both update the one committed report in parallel; disjoint section rewrites merge cleanly where a whole-file rewrite conflicts.
Bound: the splice happens only when the run's tag set equals the one in the existing header; otherwise the tool exits non-zero and writes nothing.
A section is removed only when the package lists no tests; a package that fails to list or build keeps its old section and makes the run exit non-zero, while the sections of the other measured packages are still written.
A measured package with no section in the existing report gets a new one, placed in the report's package order (sorted by import path).
A missing `-out` file is written whole.

### D4 — Deleting a candidate

Decision: a candidate is deleted only if it is in the removable set of the current report and a read of its assertions finds nothing its covering tests do not also assert.
The same read asks whether the candidate makes git run module code, through a hook, alias or helper it installs; the scan cannot see that, and such a candidate is kept under D2.
Otherwise it is folded into its covering test as an assertion or a table row, or kept under D2.
After a deletion pass the tool is rerun on the six packages and the pass repeats until no candidate outside the Kept list remains.

Rationale: `PATTERN-test-economy`: coverage blocks do not see assertions, and only the removable set may go together.

### D5 — Folding near-identical tests

Decision: tests in one package that share setup and differ only in inputs and expected outputs become one table-driven test.
Each former test survives as a named `t.Run` subtest, so its failure stays identifiable.
Folding is judged by reading the tests, including the ones the tool excludes; it applies to every test in the six packages, not only candidates.
A folded test keeps `t.Parallel` per `PATTERN-test-speed`.

Rationale: the board's second clean step, and `PATTERN-test-economy`'s rule that many items of one kind share one table-driven test.

### D6 — Moving integration tests to the untagged tier

Decision: an `integration` or `tmux` test whose behavior does not depend on real git, tmux or a process moves to an untagged file and runs against a fake or in-memory fixture.
A fake used by more than one package lives in `internal/testkit/<kit>/` (`PATTERN-testkit`); a fake used by one package stays in that package's test files.
Where the production code offers no seam for the fake, the task adds one as a struct field or option, never a package-level variable, with production passing today's value.

Rationale: `PATTERN-test-economy`: a behavior testable with a fake is tested untagged.
Bound: a seam changes no production behavior, since production passes the current dependency; the moved test passes `tierpurity_test` and `PATTERN-test-isolation`, or it stays tagged.
A package whose last `integration`, `tmux` or `llm` test file is retagged untagged may remove the `tmuxkit.Main` call from its `TestMain`, and a package that keeps any such file keeps the call.
A package that still spawns git keeps `gitkit.HermeticGitEnv()`.

### D7 — Tests the tool cannot run alone

Decision: a test the report lists as "failed" when run alone is fixed so it passes alone, or the tool's single-test run is fixed if the fault is the tool's.
The rerun lists none of the six packages' tests as "failed".

Rationale: a test that fails alone either depends on another test's side effect, which `PATTERN-test-speed`'s parallel rule forbids, or exposes a tool defect; either way the test stays outside judgment until fixed.

### D8 — Measuring before and after

Decision: before any test change, record per package the top-level test count, wall time and serial time from `go run ./cmd/testtiming -tags integration,tmux` (median of three runs) and statement coverage of the package's own code from `go test -tags integration,tmux -cover`.
Repeat the same measurements at the end, on the same machine.
The PR description carries the before and after table and the load average of each run.
These numbers are not committed to `docs/benchmarks/test-suite-timing.md`; `test-tiers` records the after-state there.

Rationale: the board's done-when asks the PR to state count and wall time, and the coverage figure checks that nothing beyond redundant coverage was lost.

### Added after Discussion (operator, 2026-10-06): Scenario tests share one expensive fixture

Decision: Where several integration or tmux tests in this task's packages each build the same expensive fixture (a git hub or pair through hubforge, a reed or tmux server, a seeded worktree) and differ only in the steps they run and the checks they make after that setup, they become one scenario test that builds the fixture once and runs the steps in order. Each former test becomes a named t.Run step inside it, so a failure still names the step that broke; a step that depends on an earlier step's state says so in a comment. Scope is bounded to such fixture families, judged by reading the tests; a family is merged only when it cuts the package's wall time, which the before and after measurement shows. Every assertion of the merged tests is kept. Cheap unit tests are not merged into scenarios; they stay table-driven.

Rationale: Operator, 2026-10-06: the suite's cost sits in the git and tmux fixtures each test rebuilds, which deleting redundant tests and folding into tables barely touches. These tasks already read every test file in their packages, so a later pass for the same files would repeat the work. Note composite-tests records the idea.

### Added after Discussion (parent, 2026-10-06): Stub driver helpers stay untouched

Decision: This task leaves writeStubDriverScript in internal/loomcli/integration_driverbootstrap_test.go and integrationWriteStubDriverScript in internal/loomcli/smoke_driverstrand_test.go unchanged: no edit, move, fold or deletion, and no change elsewhere that requires one. Tests in those files may still be pruned, folded or retagged as long as both helpers keep their names, signatures and bodies.

Rationale: The parallel task skills-in-one-turn edits both helpers; leaving them alone keeps the two PRs from conflicting.

### Added after Discussion (parent, 2026-10-06): Measurements go in a committed benchmark report

Decision: D8's before and after numbers (per package: top-level test count, wall and serial time as medians of three runs, statement coverage, and the load average of each run) are written to a new measurement report, docs/benchmarks/test-suite-prune-core.md, linked from docs/benchmarks/running-tests.md. The before half is committed before any test change, the after half at the end, together with each lost coverage block and the deleted test that alone covered it. test-suite-timing.md stays untouched.

Rationale: The Describe session that writes the PR description reads only the decision record, the board entry, the task diff and the run records, so numbers kept outside the diff never reach the PR. docs/benchmarks already holds measurement reports of this kind (fixture-copy.md), and a measurement report is allowed to record one run at a point in time.

### Added after Discussion (parent, 2026-10-06): A scenario stays within one tier

Decision: A scenario merges only tests of one tier. A merge never moves a test to a higher tier: an integration test never joins a tmux scenario.

Rationale: A higher tier is run by fewer gates, so a merge across tiers would silently take a test out of a gate that ran it.

## Constraints

- `PATTERN-test-economy`: candidate reading rules, removable-set-only deletion, folding into tables. D2's keep directive extends its background.
- `PATTERN-test-speed`: tier rules (D1 relies on "untagged spawns nothing"), injectable time and seams as fields (D6), `t.Parallel` in folded tests.
- `PATTERN-test-isolation`: hermetic git and `tmuxkit.Main` stay correct for every package whose tag set changes.
- `PATTERN-testkit`: shared fakes under `internal/testkit/<kit>/`, imported only from tests.
- `PATTERN-hubforge-fixtures`: a test that keeps a real hub builds it through `hubforge`; a moved test drops the hub, never hand-assembles one.
- `PATTERN-comment-line-breaks` and semantic line breaks in every changed comment and `.md` file.
- `PATTERN-markdown-link-integrity` for changed docs under `docs/`.
- Enforcement tests in `cmd/lyx` (`tierpurity_test.go`, `llmtier_test.go` and the guards they rely on) keep passing.
- No test in another package's files is edited; `test-suite-prune-rest` runs in parallel.

## Auto-mode assumptions

- The tool changes (D1–D3) belong to this task, because the board assigns shared test helpers to it and `test-suite-prune-rest` benefits once this lands.
- Coverage is measured under `-tags integration,tmux`, the tag set the committed report uses; `llm` files are not run.
- The keep directive's spelling is `//testtiming:keep`, a Go directive comment, so godoc does not show it.
- Test count means top-level test functions, as `cmd/testtiming` counts them; folded subtests do not count.

## Open risks

- `test-suite-prune-rest` runs in parallel against the pre-D1 tool and may also edit `cmd/testtiming` or the report header; the hub compares both PRs at PR-Gate.
- D1 raises the candidate count sharply, so the hand check under D4 is the bulk of the work.
- Wall time moves with machine load; the before and after runs may not be comparable if other runs are live, which the recorded load averages expose.
- A seam added for D6 touches production code in a package other tasks also change.

## Acceptance criteria

- `go run ./cmd/testtiming -redundancy -tags integration,tmux -pkg <the six packages>` lists, for each of the six packages, no candidate outside the "Kept" list and no test as "failed".
- Every "Kept" entry has a non-empty reason; the tool refuses a `//testtiming:keep` with an empty reason.
- A test that spawns only git in an integration file is judged by the tool; no test in an untagged file is listed with the spawns or unclassifiable reason.
- A test that references `lyxbin`, `os.Executable`, `os.Args[0]` or the `go` toolchain, or lives in a `tmux` file, is still excluded, and so is a test in a tagged file with a call the scan cannot resolve.
- A `//testtiming:keep` directive that is separated from its test, duplicated on one test, or has no test below it is refused like an empty reason.
- A narrowed rerun rewrites only the measured packages' sections of an existing report and leaves the others and the header byte-identical.
- A narrowed rerun removes the section of a package that lists no tests, refuses a tag set that differs from the header's and writes nothing, and writes a missing `-out` file whole.
- A narrowed rerun with a package that fails to list or build keeps that package's old section, still writes the other measured sections, and exits non-zero.
- A narrowed rerun for a package with no section in the existing report adds one in import-path order.
- A `//testtiming:keep` on a test that is not a candidate is listed under "Kept" with its reason, and a kept test still counts among the covering tests of the others.
- A `//testtiming:keep` with a whitespace-only reason is refused like an empty one.
- Each package's statement coverage after is at least its coverage before, or the PR names each lost block and the deleted test that alone covered it.
- The PR states, per package, top-level test count, wall time and coverage before and after.
- Every folded test keeps each former test as a named subtest.
- `go vet` under each tag, `go test ./...`, and `go test -tags integration ./...` and `-tags tmux` on the six packages and `cmd/lyx` and `cmd/testtiming` pass.
- `pattern/PATTERN-test-economy.md`, `docs/benchmarks/running-tests.md` and the `cmd/testtiming` package doc describe D1's exclusion rule, D2's directive and D3's splice.

## Notes for the plan writer

- The exclusion logic lives in `cmd/testtiming/spawnscan.go` and `redundancy.go` (`noCoverageReason`, `spawnVerdict`); report writing in `redundancy.go`.
- Card order: tool changes first, then the before measurement (it needs no tool change but must precede test edits), then the rerun report, then per-package prune cards, then the final rerun and docs.
- Per-package cards keep each card's verify to its own package plus `cmd/lyx`'s guards.
