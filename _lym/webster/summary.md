# machine-switch: `lym adopt` holds the runs it takes over and checks each subject

All six batches recorded `status: done`; the head after the last is `bb01d221`.
No digest carried `departures`, `renames` or `warnings`.
The scope guard listed only files outside the batch targets, all informational.

## Batches

- **01-hold-caller** (cards 1–4): the pause record's hold caller, dobby giving a held run no pass, the board's `front` and `on_land` fields, and an in-process batten cancel with a matchable nothing-left refusal.
  Deviations: `internal/battencli/cancel.go`, `internal/boardengine/front.go`.
- **05-adopt-check-per-subject** (card 5): one adopt check per subject, in its worktree, with per-subject files and view.
  Deviations: `internal/adoptcli/check.go`, `internal/adoptcli/checkview_test.go`.
- **06-front-task-package** (cards 6–7): `internal/fronttask`, and dobby starting an open front task first and settling its land action.
  Deviations: `internal/battencli/dobbyreads.go`, `internal/battencli/overviewdobby.go`, `internal/dobbyengine/loop.go`, `starts.go`, `starts_test.go`, `view.go`, `internal/fronttask/fronttask.go`, `internal/fronttask/hub.go`.
- **08-darn-condition-stop** (cards 8–9): the darn writer's stop on a condition the task body names, and beam.yaml's `adopt_fix_tries`.
  Deviation: `internal/darnengine/spec_test.go`.
- **10-task-branch-slugs** (cards 10–11): fabric lists task branches of a slug prefix, and adopt holds the runs it takes over before the owner write and resumes none.
  Deviations: `internal/adoptcli/adopt.go`, `adopt_test.go`, `holds.go`, `internal/gitrepo/refreads.go`.
- **12-check-green-end** (cards 12–14): a green check lifts holds, starts the runs and removes the fix task; a failed check files the fix task up to the try limit; adopt reports one line per run.
  Deviations: `internal/adoptcli/adopt.go`, `adopt_test.go`, `fixloop.go`, `fixloop_test.go`, `green.go`, `green_test.go`, `runreport.go`, `runreport_test.go`.

The forks reported their batch gates green on tiers 0 and 1; tiers 2 and 3 did not run.

## Verify gate findings

Attempt 1 of the plan-level verify failed in package `github.com/Knatte18/loomyard/cmd/lym`; the findings file notes that no recorded card touched a failing package.
A fixer fork then ran and committed one fix.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `d9d4df2b8c58521f5085e208d115a3946b781f0b` fix: keep the adopt and board upsert help pages within the Long cap
