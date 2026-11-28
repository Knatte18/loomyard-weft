# Run the battencli rows scenario in parallel groups, a hub each

`TestBattenIntegration_Rows` ran all of its steps one after another on one hub copy,
so the whole scenario set the floor of the `internal/battencli` integration test run.
Its steps now sit in seven groups, each over a hub of its own, and the groups run in parallel.

## What changed

- `internal/battencli/lifecycle_integration_test.go`: the flat step table becomes a table of groups.
  Each group subtest calls `t.Parallel`, copies its own hub and runs its steps in order, ending the group at its first failed step.
  A failed step no longer stops the other groups.
  The groups are `SeedChild`, `FourRowRun`, `TeardownRemote`, `TeardownLocal`, `RunShed`, `CreateRow` and `CommandRows`.
  The step functions and their assertions are unchanged.
- The same file: the step that captures the process-global logger, `stepCommandRows_IgnoreACommandSetInTheTasksTrackedLayer`,
  leaves the table for a serial test of its own,
  `TestBattenIntegration_CommandRowsIgnoreACommandSetInTheTasksTrackedLayer`.
- `pattern/PATTERN-pair-remote.md`: the covering subtest it cites is now named
  `TestBattenIntegration_Rows/CreateRow/RecordsPrimeRefusal`.

## Design decisions

- Steps stay sequential inside a group because they share its hub.
  The step that dirties the prime worktree runs last in `CreateRow`.
  The steps that break the records origin on purpose (a deletion-refusing hook, an unreachable URL) share `TeardownRemote`,
  and the grouped steps that rewrite the board default `batten.yaml` commands share `CommandRows`.
  The log-capturing step rewrites them too, on the hub of its own serial test.
- The cancel tests keep their structure.

## Measurements

Each run was `lym gate test --gate darn-write ./internal/battencli -- -count=1`,
with `--no-carry` added on every run after the first.
Each package figure is the `internal/battencli` time of that call's tier1 phase.
The six runs ran on the same 12-thread machine within four minutes, three before the change and then three after it.

| Run | Package before | `TestBattenIntegration_Rows` before | Package after | Longest group after |
|---|---|---|---|---|
| 1 | 11.39 s | 7.67 s | 6.14 s | 1.69 s (`TeardownLocal`) |
| 2 | 9.71 s | 7.46 s | 5.54 s | 2.07 s (`TeardownLocal`) |
| 3 | 9.32 s | 7.09 s | 7.48 s | 2.61 s (`TeardownLocal`) |

The after runs predate the move of the log-capturing step into its own serial test.

The 33 to 75 s in `docs/research/speed/test-time-measure-2026-11-27/report.md` were measured
under a 1-minute load average of 23 to 67 with other test runs holding slots.
These six runs ran under a much lighter load: `sar -q` records a 1-minute load average of 4.9 at 22:20 and 3.6 at 22:30,
the latter between the two sets of runs (22:27 to 22:28 and 22:30 to 22:31).
No sample falls within either set.

## To check by hand

- The package times after the change were not re-measured once the log-capturing step became a serial test.
  Run the command above three times on the final code to confirm the after figures.

## Merge danger

**Door:** two-way
**Blast radius:** tests
