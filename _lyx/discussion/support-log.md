# Support log — test-suite-prune-core

## Interview

Autonomous mode: no operator answered, so every turn below is a self-pick from the board entry, the related entries and the code.

1. Scope: the board entry names the six packages and three clean steps; `test-suite-prune-rest` takes every other package, and `test-tiers` takes the after-state, the budget guard and gate selection.
   Picked: test files of the six packages, plus shared test helpers.
2. Is the tool fit for the task? The committed report puts most tests of the six packages under "no coverage": 602 of 697 in fabricengine, 212 of 259 in shedadapters, 181 of 213 in shuttleengine.
   The cause is the spawn scan: any test reaching git, or any call it cannot resolve, is excluded.
   Picked: narrow the exclusion to tests that may run module code out of process (D1); without it the task prunes only a fraction of each package.
3. Where does a kept candidate's reason live? The done-when wants the tool's rerun to show no candidate without a reason.
   Picked: a `//testtiming:keep <reason>` directive the tool reads (D2).
4. Parallel edits to one report: both prune tasks regenerate `docs/benchmarks/test-redundancy.md`.
   Picked: a narrowed run rewrites only its own sections (D3).
5. Deletion rule, folding rule and the integration-to-untagged move follow `PATTERN-test-economy`, `PATTERN-test-speed` and `PATTERN-testkit` directly (D4–D6).
6. Nine loomcli tests are listed "failed" when run alone. Picked: fix them or the tool so they are judged (D7).
7. Measurement: the PR carries count, wall time and coverage per package before and after; the benchmark doc's after-state is `test-tiers`' (D8).

## Rejected alternatives

- Keep the tool as is and prune only the current candidates: leaves 70–98% of each package's tests unjudged, so the task would fall back on reading thousands of tests with no coverage evidence.
- State keep reasons only in the PR description: a rerun of the tool cannot see them, so the done-when stays unverifiable after the PR is merged.
- Keep reasons in a sidecar file beside the report: a second file to keep in sync with test renames; the directive moves with the test.
- Regenerate the whole report in this task: conflicts with `test-suite-prune-rest`'s parallel regeneration.
- Leave the tool changes to `test-suite-prune-rest` or a new task: the board assigns shared test tooling here, and this task needs the change first.
- Record the after-state in `docs/benchmarks/test-suite-timing.md` here: `test-tiers` owns the after-state, measured once both prunes have landed.
- Treat every spawn as excluding: git runs no module code, so it hides no coverage.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Scope of `cmd/testtiming` changes — resolved, auto-pick: in scope (D1–D3), because shared test helpers belong to this task and the tool cannot judge most of these packages as it is.
- Q2 Spelling and placement of the keep marker — resolved, auto-pick: `//testtiming:keep <reason>` above the test function.
- Q3 Report merge strategy with `test-suite-prune-rest` — resolved, auto-pick: section-splicing write (D3); the hub compares the PRs at PR-Gate.
- Q4 Tag set for coverage and timing — resolved, auto-pick: `integration,tmux`, matching the committed report.
- Q5 Where before/after numbers go — resolved, auto-pick: the PR description only; the benchmark doc's after-state is `test-tiers`'.
- Q6 Production seams for moved tests — resolved, auto-pick: allowed as struct fields or options in the six packages only, production passing today's value.
- Q7 Loomcli tests that fail alone — resolved, auto-pick: fix the test or the tool so each is judged.
- Q8 Open: whether `test-suite-prune-rest` also edits `cmd/testtiming`; left to the hub's PR-Gate comparison.
