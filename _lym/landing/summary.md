# Report on tier 1: where a pass goes and the ranked changes to cut it

Adds `docs/research/speed/tier1-test-revision-2026-11-06/report.md`, a ranked report on what to change in tier 1, and an index line for it in `docs/research/README.md`.
It changes no code, test or config and files no task, as the board entry asked.
The report builds on the 2026-11-03 fabricengine survey and the 2026-11-06 run-slowness report, and checks the survey's levers against today's tree rather than repeating its measuring.

## What the report finds

- A cold whole-repository tier 1 pass takes 75.0 s on a quiet neve, and a median 111 s over the 43 whole passes the gate recorded since 2026-11-05.
  `internal/fabricengine` is its wall at 57.0 s, after a floor of about 18 s of build, vet, link and binary starts.
- 130 of the 196 test binaries a tier 1 pass starts hold no tier 1 test.
  Starting only the 66 that do takes the floor from 20.5 s to 12.4 s, measured through the gate: 8 s off every pass, with no test lost.
- None of the survey's levers has landed, and all still stand.
  The ranked list in the report puts registered hub templates first by saving (about 10 s of the pass), then `worktree list` through `dotgit` (5 s, one function with 35 callers), then smaller changes; together they take `internal/fabricengine` from 57 s to about 36 s and a pass from 75 s to about 47 s with the binary list.
- A pair template costs about 0.18 s to build once per test binary and saves 67 ms per pair a test adds, measured with a throwaway `hubforge` probe whose source sits beside the report.
  The closed `Shapes` set cannot hold the start states the fixtures need, so the lever is a template registered with a builder function.
- Folding overlapping tests is the smallest lever, about 7 s of 156.8 s serial, and the coverage redundancy report finds five removable tier 1 tests worth 1.4 s over twelve packages.
  `TestVerbCases_CleanState` repeating the clean cells of `TestCrossProduct` is the one fold worth its own task.
- The report answers all seven questions: the pass and its packages, the fabricengine levers and the template cost, the other slow packages, folds and tier 0 moves, sharing one repository in the three shapes, the 26,000 reads that start git and could be answered in process, and the before and after of a darn run's gate.
  It ends with ten ranked follow-ups, each with saving, size and risk to coverage.

## Design decisions

- **A fourth whole-repository run.**
  The operator's bound was three whole-repository tier 1 runs.
  The measuring made four: W1, W2 and W3, and a repeat of W2, which was a mistake and agrees with W2 to 0.1 s.
  The report's route section states the overrun.
  Two runs of `internal/fabricengine` were made, as bounded.
  A 66-package run and a hubforge probe run were made through the gate; neither is whole-repository, and the twelve-package redundancy report ran `go test` through `cmd/testtiming`, never over `internal/fabricengine`.
- **The redundancy report was run on twelve packages and not on `internal/fabricengine`.**
  Its per-test coverage runs would have been a third tier 1 run of the package.
  The fabricengine fold candidates come from reading, as the survey's did; three were read again today and the rest are the survey's rows with today's costs, and the report says which.
- **Per-test seconds come from the gate's text logs.**
  A log interleaves packages, so `pertest.sh` ties a test to its package by the one `func Test` that declares it, and the report states that a test's cost sums its parallel subtests.
  Survey and report per-test numbers are not comparable: the survey's came from a traced run.
- **The darn run's after figures are a model.**
  The report states the model and its basis: the critical path while `internal/fabricengine` is the wall, total CPU under load.
- **Ranking criterion.**
  The package levers are ranked by wall saving; the follow-up tasks are ranked by saving per unit of size, so the measured and small ones precede the large template task that saves most.

## Verification

No code changed, so no test applies.
`pertest.sh` was run against the serial run's log and reproduces its package total, 156.6 s.
The probe package and the marker file used while measuring were removed before the commit.
