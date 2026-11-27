# Run the battencli rows scenario in parallel groups, a hub each

`TestBattenIntegration_Rows` ran all of its steps one after another on one hub copy,
so the whole scenario set the floor of the `internal/battencli` integration test run.
Its steps now sit in seven groups, and each group runs over a hub of its own; the groups run in parallel.

## What changed

- `internal/battencli/lifecycle_integration_test.go`: the flat step table becomes a table of groups.
  Each group subtest calls `t.Parallel`, copies its own hub, and runs its steps in order, ending the group at the first failed step.
  The groups are `SeedChild`, `FourRowRun`, `TeardownRemote`, `TeardownLocal`, `RunShed`, `CreateRow` and `CommandRows`.
  Every step function and every assertion is unchanged; only the table and the test's doc comment moved.
- `pattern/PATTERN-pair-remote.md`: the subtest that covers the records-prime refusal is now named
  `TestBattenIntegration_Rows/CreateRow/RecordsPrimeRefusal`.

## Design decisions

- Steps stay sequential inside a group, so the ordering rules the old table relied on still hold.
  The step that dirties the prime worktree runs last in `CreateRow`.
  The steps that break the records origin on purpose (a deletion-refusing hook, an unreachable URL) share `TeardownRemote`,
  and the steps that rewrite the board default `batten.yaml` commands share `CommandRows`.
- Groups were balanced on per-step times from the runs before the change, aiming at one to one and a half seconds of step time each.
- The cancel tests are left as they are.

## Measurements

Each figure is the `internal/battencli` package time `go test -count=1` reports for the integration test run.
All six runs ran back to back on the same machine.

| Run | Package before | `TestBattenIntegration_Rows` before | Package after | Longest group after |
|---|---|---|---|---|
| 1 | 11.39 s | 7.67 s | 6.14 s | 1.69 s (`TeardownLocal`) |
| 2 | 9.71 s | 7.46 s | 5.54 s | 2.07 s (`TeardownLocal`) |
| 3 | 9.32 s | 7.09 s | 7.48 s | 2.61 s (`TeardownLocal`) |

The machine ran these well below the 33 to 75 s the earlier measurement recorded, so the absolute figures differ from it.
After the change the package's longest test is `TestCancelIntegration_RemoveOfABeamLeavesAPlainProject` or a rows group, at 1.9 to 2.6 s.

## Merge danger

**Door:** two-way
**Blast radius:** tests
