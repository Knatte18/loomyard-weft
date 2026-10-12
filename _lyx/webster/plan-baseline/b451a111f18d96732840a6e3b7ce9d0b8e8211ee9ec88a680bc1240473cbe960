# Card 15 — report-cli

**Create:**
- `plan:internal/reportcli#Command` -> `func Command() *cobra.Command`
- `plan:internal/reportcli#RunCLI` -> `func RunCLI(out io.Writer, args []string) int`
- `plan:internal/reportcli#RunCLIIn` -> `func RunCLIIn(cwd string, out io.Writer, args []string) int`
- `plan:internal/reportcli#refuseNonPrime` -> `func refuseNonPrime(worktreeName, primeName string, primeNameErr error) error`
- `//internal/reportcli/doc.go`
- `//internal/reportcli/cli_test.go`
- `//internal/reportcli/convergence_integration_test.go`
- `//internal/reportcli/testmain_test.go`

**Edit:**
- `cmd/lyx#newRoot`
- `//cmd/lyx/seamsignature_test.go`
- `//cmd/lyx/testdata/test-budget.yaml`
- `//tools/sandbox/SANDBOX-CORE-SUITE.md`
- `//docs/overview.md`

**Uses:**
- `internal/reportengine#Open`
- `internal/reportengine#Compute`
- `plan:internal/reportengine#RenderMarkdown`
- `internal/fabricengine#RecordsRepoRoot`
- `internal/fabricengine#RecordsBranchName`
- `internal/fabricengine#BoardDir`
- `internal/fabricengine#PrimeName`
- `internal/fabricengine#RequireDrivableWorktree`

**Intent:**
D1 mounts `lyx report convergence` under the root, and D2 runs it from the hub's prime worktree only, the CLI resolving the geometry it tells the engine.

`internal/reportcli` (in `cli.go` and `refusal.go`) follows `PATTERN-cli-cobra` and imports `reportengine`, not another cli package.
`Command` returns the bare `report` group: `RunE: clihelp.GroupRunE`, no audience, a non-empty `Short`.
Its one subcommand, `convergence`, carries `clihelp.AudienceOperator`, a one-line lower-case `Short`, a `Long` under the clitree byte cap naming the four board questions, the flags and the prime-only rule, and `cobra.NoArgs`.
Its `RunE` checks `clihelp.ShouldAbort` first, then, before any read:
- parses `--since` as `2006-01-02` in UTC, defaulting to the UTC date twelve weeks before now;
- requires `--fresh-days` (default 3) to be at least 1;
- requires `--format` (default `json`) to be `json` or `markdown`.
Each refusal is a JSON error naming the flag, the value given and the form accepted, with exit 1 (D4).
It then resolves the cwd through `lyxcwd.CwdFrom` and `lyxcwd.Resolve`, refuses outside the prime with `fabricengine.RequireDrivableWorktree` and `refuseNonPrime` (orch's shape with report's wording: the worktree is not the prime, the prime's name, re-run it there), and builds `reportengine.Geometry`:
`CodeRepo` the prime's worktree path, `ParentBranch` the prime's current branch read through `gitrepo.Repo.CurrentBranch`, `RecordsRepo` `fabricengine.RecordsRepoRoot`, `RecordsBranch` `fabricengine.RecordsBranchName(ParentBranch)`, `BoardRepo` `fabricengine.BoardDir(hub)` and `BoardBranch` that checkout's current branch.
It calls `reportengine.Open` and `reportengine.Compute`; an error from either is a JSON error with exit 1.
With `json` it prints the report's fields through `output.Ok`, and with `markdown` it writes `RenderMarkdown`'s text; both exit 0 whatever findings the report holds (D5), and a hub with no archive tags prints an empty report.
The verb writes nothing, takes no lock and makes no commit (D3).
What it newly lets through is a read of the hub's git history; an explicit early `--since` widens blame cost linearly with the landings it selects, which the operator accepts by passing it (D4), and the prime-only refusal bounds where it runs.

`cmd/lyx#newRoot` mounts `reportcli.Command()`; `seamsignature_test.go` adds `reportcli.RunCLI` and `reportcli.RunCLIIn` to its two pinned lists.
`tools/sandbox/SANDBOX-CORE-SUITE.md` gains `### S12 -- Convergence report from the prime` with `**Covers:** report`: from the sandbox's prime, `lyx report convergence` prints one `{"ok":true}` envelope with `runs`, `weeks` and `findings`; `--format markdown` prints the four section headings; `--fresh-days 0` refuses naming the flag; run from a task worktree it refuses and names the prime; and the session log format gains its `S12:` line.
`docs/overview.md` gains `internal/reportcli/` and `internal/reportengine/` rows in its module tree and a **report** bullet beside **cleanup**: the read-only convergence report (`lyx report convergence [--since <date>] [--fresh-days <n>] [--format json|markdown]`, prime only), pointing at `internal/reportengine`'s package documentation.
`doc.go` documents the module, the flags, the refusals and the output.

Tests:
- `TestConvergence_RefusesBadFlags` (in `cli_test.go`, untagged) runs `RunCLIIn` from `t.TempDir()`, which is no git checkout, with rows `--since 2026-13-01`, `--since yesterday`, `--fresh-days 0` and `--format yaml`; each exits 1 with an envelope whose error names the flag, which shows the refusal comes before any cwd resolution or read.
- `TestRefuseNonPrime` (in `cli_test.go`, untagged) is a table over `refuseNonPrime`, the shape of `internal/orchcli/cli_test.go`'s: the prime's own name passes, another name is refused naming the prime, and a non-nil `primeNameErr` is an error that wraps it.
- `TestConvergence_RefusesOutsideThePrimeThenReportsTheSeededRun` (in `convergence_integration_test.go`, `//go:build integration`) builds `hubforge.CopyHub(t, hubforge.Shape{Anchor: ".", Pairs: []string{hubforge.TemplatePairSlug}})` and, in order:
  run from the prime's records-side worktree (`fabricengine.RecordsRepoRoot` of the prime's location), it exits 1 with the `RequireDrivableWorktree` refusal;
  run from the pair's worktree, it exits 1 and its error names the prime;
  run from the prime before any tag, it exits 0 with no runs;
  it then seeds one finished run of the template pair, in this order: a new code commit in the pair's worktree with `gitkit.Git`, so the pair's code HEAD is a commit the prime's branch does not hold; the pair's loom seed (recipe `loom`), a status whose history holds `Finalize` `done`, and a decision record whose one `### Added after Discussion (` heading carries no date, committed on the pair's records side through `Fabric.Commit` on `fabricengine.Open` of the pair's location (from `lyxcwd.Resolve(h.PairCodeWorktree(slug))`), the commit path that writes the task-commit trailer (`CommitAnchoredPaths` writes none), so that commit's trailer names the new code commit; a batten `status.json` for the slug on the prime's records branch whose history runs from `Worktree-Create` to `Land` `done`; a board entry for the slug labelled `bug` through `boardengine.OpenHub` and `UpsertTask`; the new code commit fast-forwarded onto the prime's branch with `gitkit.Git`; and an `archive/<slug>/<sha12>` tag at the pair's records tip, also with `gitkit.Git`;
  run from the prime with no `--since`, so the twelve-week default applies, and with `--fresh-days 1`, it exits 0 with `selection.since` equal to one of the two dates 84 days before the UTC dates the test takes just before and just after the `RunCLIIn` call, one run whose batten source is present, exactly one landing for the slug, of kind `fix` and joined `exact` to that run (the hub template's own commits, dated today, are in range and read `unmapped`, which the test does not count), and one finding naming the decision record, which pins that a finding leaves the exit code 0 (D5);
  run with `--format markdown`, it prints the four section headings.
- `testmain_test.go` runs the package under `tmuxkit.Main`.

`cmd/lyx/testdata/test-budget.yaml` gains `internal/reportcli: 3`.

**ImpactSummary:** `newRoot` mounts one more module, which `TestRegistration_AllModulesRegistered`, the clitree walk and `TestSandboxCoverage_AllModulesCoveredOrExcluded` then check; the rest is new files, docs and one test list.

**Commit:** `15: lyx report convergence`
