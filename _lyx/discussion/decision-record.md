## Goal

Shrink the test suite in every Go package outside `test-suite-prune-core`'s set without losing a pinned contract.
Delete the tests that `docs/benchmarks/test-redundancy.md` reports as redundant once each one is checked, fold near-identical tests into table-driven tests, and move `integration` tests that spawn nothing down to the untagged tier.
The PR reports per-package test count and wall time before and after, and shows coverage has not dropped beyond what the deletions explain.

## Scope

In scope: every package in `docs/benchmarks/test-redundancy.md` except the six core packages and `internal/testkit/...`.
That covers `cmd/...`, `contracts/...`, `tools/...` and every `internal/...` package outside the excluded set.
That includes `internal/reedengine/render`, `internal/shuttleengine/claudeengine` and `internal/gitrepo/internal/gitoracle`, which are separate Go packages, not part of their parent's directory set.

Out of scope:

- `internal/fabricengine`, `internal/websterengine`, `internal/reedengine`, `internal/shuttleengine`, `internal/loomcli`, `internal/shedadapters` (owned by `test-suite-prune-core`).
- Every file under `internal/testkit/` (shared helpers and the kits' own tests belong to `test-suite-prune-core`).
- `llm`-tagged test files: the report did not measure them, and running them costs tokens.
- Production code, except deleting a non-test helper that pruning leaves with no caller (none expected).
- `cmd/testtiming`'s behavior: the tool stays as `test-suite-measure` built it.
- The after-state in `docs/benchmarks/test-suite-timing.md`, the budget guard and gate selection (all `test-tiers`).

## Decisions

1. **Ownership boundary.**
   This task edits no file under the six core package directories or under `internal/testkit/`.
   Rationale: the two runs work in parallel, and disjoint file sets keep them mergeable without hub arbitration.
   A file in an in-scope package that names a core file stays as is unless an in-scope change requires an edit.
   Example: a guard allowlist in `cmd/lyx` listing a path under a core package.

2. **Deletion rule.**
   A candidate is deleted only when the report marks it `Removable: yes`, and only after its assertions are read against those of its covering tests.
   A candidate whose assertions pin something its covering tests do not pin is handled in one of two ways.
   Either that assertion moves into a covering test (a new row or an extra check) and the candidate is deleted, or the candidate is kept with a stated reason.
   Examples of such assertions: an error message, an output or envelope shape, a refusal's way-forward text, a file's exact contents.
   A `Removable: no` candidate is never deleted on the report's evidence alone; it may still be folded (Decision 4).
   Rationale: coverage blocks do not see assertions (`PATTERN-test-economy`), and only the removable set is safe to delete together.

3. **Guard self-checks are kept.**
   Some candidates exist to prove that a repo-scanning guard fires on an offending input, such as `*_NotVacuous`, `*_DetectionHelper` and `*FiresOnOffendingStaysQuietOnCompliant` in `cmd/lyx`.
   These tests pin a contract the coverage view cannot see: on a compliant tree, the guard's scan covers the same blocks but asserts nothing about firing.
   Each is kept with that reason, or folded into a table with its siblings if they share a shape.
   Rationale: deleting them would leave the guards able to pass vacuously.

4. **Folding.**
   Each in-scope package's test files are read for groups of top-level tests that differ only in inputs and expected values.
   Each such group becomes one table-driven test with `t.Run` subtests, named after the behavior.
   The candidates and their covering tests come first, then the rest of the package.
   A fold keeps every assertion of the tests it replaces.
   A folded table follows `PATTERN-test-speed`: `t.Parallel` at the top level and per subtest, unless a comment names the global state.
   Rationale: the board's Clean list asks for it in every package, and `PATTERN-test-economy` requires one table for many items of one kind.

5. **Tier moves.**
   A test in an `integration` file moves to an untagged file only when it needs neither git, tmux nor a subprocess, and passes `cmd/lyx/tierpurity_test.go`.
   Leads come from the report: tests in `integration` files that the tool measured, as opposed to those listed `spawns`.
   Each lead is confirmed by reading the test.
   A moved test uses an existing `internal/testkit` fake, or a fake that only its own package uses, local to that package's `_test.go` files (`PATTERN-testkit`).
   A move that needs a new shared fake is not made.
   The test stays where it is with that reason stated, and the hub is told so `test-suite-prune-core` can add the kit.
   Rationale: `internal/testkit` is core's (Decision 1), and duplicating a fake across packages breaks `PATTERN-testkit`.

6. **Leftovers from deletion.**
   A test helper that pruning leaves unused is deleted.
   A test file left with no tests or helpers is deleted.
   A `TestMain` stays even when its package loses its last tagged file.
   Rationale: dead helpers are noise.
   A spare `tmuxkit.Main` is harmless, and removing it would have to be undone when a tagged test returns.

7. **References to renamed or deleted tests.**
   Folding or deleting a test that a doc, a `pattern/` background, a stencil, a spec or a code comment names updates that reference in the same commit.
   Rationale: `PATTERN-documentation-lifecycle` and the docs-in-the-same-commit rule in `CLAUDE.md`.

8. **"failed" entries.**
   Each in-scope test the report lists as `failed` is re-run alone with `go test -tags integration,tmux -run '^Name$'`.
   If it fails alone, it has a hidden order or environment dependence, which this task fixes in the test.
   If it passes alone, the failure is a tool artifact, and the PR records it as such.
   Rationale: a test that fails in isolation is broken, and the tool cannot judge its redundancy until it runs.

9. **Report rerun scoped to this task's packages.**
   After pruning, `go run ./cmd/testtiming -redundancy -tags integration,tmux -pkg <in-scope packages> -out <scratch path>` is run.
   Its package sections replace those packages' sections in `docs/benchmarks/test-redundancy.md`.
   The header and the core packages' sections are left untouched.
   Bound: the edit replaces only in-scope `## <package>` sections and adds or removes none.
   Rationale: core regenerates its own sections in parallel, and each package's redundancy is judged against its own tests only, so disjoint sections merge cleanly.

10. **Stated reasons live on the test, as `//testtiming:keep <reason>`.**
    Every candidate the rerun still lists carries a `//testtiming:keep <reason>` directive comment directly above its `func Test…` line.
    The spelling is the one `test-suite-prune-core` defines in its D2.
    The reason is non-empty and names what the test pins that its covering tests do not, for example a guard that fires, an error message, an output shape, or a kept partner of a removed test.
    The PR body also groups the reasons by package and by reason class.
    This task makes no tool change: the tool at this task's base ignores the comment, and the task is judged against that tool.
    Rationale: core's tool reads the directive and lists the test under "Kept" once both tasks land, so the done-when keeps holding on `main`.
    A reason only in the PR body would show every kept candidate as unexplained on the first rerun after landing.

11. **Measurement method.**
    Before and after are measured on the same machine with `go run ./cmd/testtiming -tags integration,tmux`, median of three runs, per in-scope package.
    Before is measured on this branch's base commit, after on the final tree.
    The PR records the load average at both points.
    `-tags integration,tmux` includes untagged files and excludes `llm`, so the count covers every test this task can change.
    Rationale: the report's numbers come from coverage-instrumented binaries and are not timing data.

12. **Coverage check.**
    Per in-scope package, statement coverage of non-test code is taken under `-tags integration,tmux` with `-coverprofile`, before and after.
    A package whose coverage drops is named in the PR with the deleted or moved tests that explain the drop.
    An unexplained drop is restored by bringing back or extending a test.
    Rationale: a `Removable: yes` deletion leaves every block covered by construction, so any drop points at a mistake.

## Constraints

- `PATTERN-test-economy`: candidates are evidence, not verdicts; only the removable set is deleted together; a finding names its covering test.
- `PATTERN-test-speed`: tier tags, untagged files spawn nothing, `t.Parallel` unless a comment names the global state, no waiting out production durations.
- `PATTERN-test-isolation`: git-spawning packages keep `gitkit.HermeticGitEnv()`, and tagged packages keep `tmuxkit.Main`.
- `PATTERN-testkit`: no duplicated shared fake; a fake used by one package stays local to it.
- `PATTERN-hubforge-fixtures`: a hub fixture that stays is not rewritten into a hand-assembled one during a fold.
- `PATTERN-comment-line-breaks`: comments written or changed during folding use semantic line breaks.
- `PATTERN-markdown-link-integrity`: the edits to `docs/benchmarks/test-redundancy.md` keep every link resolving.
- `PATTERN-sandbox-coverage`: `tools/sandbox` tests that are folded or deleted still exercise or explicitly exclude every module.
- Build needs `CGO_ENABLED=1` (`PATTERN-quarry-cgo`).
- `go vet` passes under each tag on its own: none, `integration`, `tmux`, `llm`.
- No PATTERN entry changes; no new invariant.

## Auto-mode assumptions

- `internal/reedengine/render`, `internal/shuttleengine/claudeengine` and other sub-packages of core directories are this task's, because the board names packages, not directory trees.
- `internal/testkit/...` is core's in full, including the kits' own tests, from the board's "shared test helpers such as `internal/testkit` belong to this task" on `test-suite-prune-core`.
- `llm` files stay out because no automatic run checks them and the report did not measure them.
- Folding covers every in-scope package, not only the candidates, because the board's Clean list says "in every package".

## Open risks

- Coverage blocks miss assertions, so judging each candidate is manual.
  A careless deletion can drop a pinned message, and only the per-candidate assertion read (Decision 2) guards against it.
- Folding can drop an assertion silently.
  The coverage check (Decision 12) does not catch a lost assertion when another test still covers the block.
- `docs/benchmarks/test-redundancy.md` and any `cmd/lyx` allowlist naming core files are touched by both runs.
  Section-scoped edits (Decision 9) and Decision 1 keep the hunks apart, but the second run to land may still have to rebase.
- Wall time moves with machine load, especially while other loom runs are live, so the before/after per-package times are noisy.
- `test-suite-prune-core`'s D1 widens what the tool judges: tests that spawn only git, and every test in an untagged file, stop being excluded.
  After both tasks land, a rerun over this task's packages will list candidates this task's report never showed.
  Those are not this task's to handle.
- Some in-scope "failed" entries may turn out to be real order dependences, which widens the fix work (Decision 8).

## Acceptance criteria

- Every candidate that a rerun of `cmd/testtiming -redundancy` on the in-scope packages still lists carries a non-empty `//testtiming:keep <reason>` directly above its `func Test…` line.
- No test in the six core packages or under `internal/testkit/` is changed.
- Every deleted candidate was `Removable: yes`.
  Each one's unique assertions, if any, live on in a covering test.
- Guard self-check tests in `cmd/lyx` still prove their guard fires on an offending input.
- Each moved test sits in an untagged file, and `cmd/lyx/tierpurity_test.go` passes.
- The PR states test count and wall time per in-scope package, before and after.
- The PR states non-test statement coverage per in-scope package, before and after, and explains every drop.
- Each in-scope `failed` entry is either fixed or recorded as a tool artifact.
- `docs/benchmarks/test-redundancy.md`: in-scope sections regenerated, core sections and header unchanged.
- `go build ./...` passes.
- `go vet ./...` passes, as does `go vet -tags X ./...` for each of `integration`, `tmux` and `llm`.
- `go test ./...` passes.
- `go test -tags integration,tmux` passes on the in-scope packages.
- No doc, background, stencil, spec or comment names a test this task deleted or renamed.

## Notes for the plan writer

- Report reading rules: the header of `docs/benchmarks/test-redundancy.md` and `pattern/PATTERN-test-economy.md`.
- Tally at planning time with an `awk` over the report's tables: count `| yes |` rows per `## ` section and exclude core.
  The largest in-scope packages by removable count are in the parser, logger, yaml, landing, claudeengine, batten and board groups.
- Guard tests and their self-checks live in `cmd/lyx/*_test.go`; tier purity is `cmd/lyx/tierpurity_test.go`.
- Cards split naturally per package or package group.
  Measure before-state as the first step on the untouched tree.
