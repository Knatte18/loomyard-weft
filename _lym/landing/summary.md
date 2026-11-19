# Add the mutation gate report step and feed survivors to code review

The mutation trial replayed 20 landed diffs with `lym gate mutate`.
It ran 430 mutants: 374 killed, 44 survived, 12 unjudged.
26 of the 44 survivors were checks a reviewer would act on, spread over 11 of the 20 diffs.
Runs took 30 s at the median and 80 s at most.
Its verdict was to build the step after three bound changes; this change builds both.

## What changes

- A `--gate` key whose `reports` lists `mutation` now runs a mutation report step after its other report steps.
  It runs the refresh and mutation check of `lym gate mutate` over the diff from the reach step's base.
  It writes `mutation.json` in the worktree's gate directory through `internal/state`.
  It prints one stderr line with the mutants run, killed, survived and unjudged,
  the uncovered changed blocks, the packages not judged, any error lines, and the report's path.
- The step is advisory, as the reach and redundancy steps are: every failure becomes an error line and the exit code never changes.
  A call with a red run, a call outside every hub, a hub whose `scoper` is not `go`, or a task worktree with no parent branch
  writes a report that judges nothing.
- The template's `webster-fix`, `darn-write`, `code-burler-fix` and `pr-review` keys list `mutation` in `reports`.
- The code review rubric has a new `mutation_report` marker, filled by `internal/loomcli/mutationnote.go`.
  The note lists the counts and each survivor with its possible killers, but only when the report matches HEAD's tree.
  Otherwise it says no current report exists and why, and the rubric tells the reviewer to raise no finding about its absence.
  A survivor is a candidate for a finding, never a finding by itself.
- `MutationReportMatches` matches a report against a commit by the changed functions and the hashes of the Go files of their packages,
  as `ReachReportMatches` does.
  It never matches a report that judged nothing, a red-gate report or a prime report.

## The three trial changes

1. **Spread over functions.**
   `mutation_max_mutants` still takes one mutant per changed function in turn across the packages.
   When a round has more functions than the cap leaves, it now takes functions spread evenly over the order of package, file and name,
   instead of the first ones only.
2. **Skip noise sites** (`internal/testcover`).
   A mutation site inside a statement that only logs gets no line mutant.
   That covers a call into the logger package, and an `if` with no init statement and no call in its condition whose every branch only logs.
   So does a site inside a `==` or `!=` comparison of `runtime.GOOS` with the name of a platform other than the build's.
   This replaces the old rule that skipped a dropped logger call only.
3. **One summary line, full list in `mutation.json`.**
   The report holds every survivor, unjudged mutant, uncovered changed block and package not judged.

## Design decisions

- The step reuses `reach_budget_min`, `mutation_max_mutants`, `mutation_budget_min` and the memory guard, with no new config key.
  The template comments and `internal/gateslot` docs now name the step beside `lym gate mutate`.
- A `Config.HasGoCoverageMaps` method says whether the scoper works on Go packages, so a step that reads Go coverage maps can judge nothing otherwise.
- Inside an enclosing gate run's slot the step takes no slot of its own, and its slot holder record carries its own label.
- Docs are updated in `internal/gatecli/doc.go`, `docs/overview.md`, `pattern/PATTERN-test-speed.md` and `pattern/PATTERN-coverage-map-reader.md`.

## To check by hand

- Run `lym gate test --gate darn-write` on a real multi-function diff.
  Check that the summary line reads sensibly and that `mutation.json` lists the survivors.
  The step's unit and integration tests cover this, but no live gate call was run for this description.
- The step adds mutant builds and runs to `webster-fix` and `pr-review`, which carried no report step before.
  Confirm the added time, bounded by `mutation_budget_min` and `reach_budget_min`, suits those keys.
- Bound: the file hashes cover Go files only, so a change to a non-Go file a package's tests read, such as testdata,
  leaves the report matching HEAD.
