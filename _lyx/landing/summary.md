# tokencount reports cost per segment, most expensive parts first

The tokencount report now shows at a glance where a run's money goes.
It used to list one flat row per role and per `+sub` role, ranked by token weight, so the cost of one segment meant adding up several rows by hand.

## What the report shows

- **Most expensive parts** opens the report body: the ten costliest parts summed over all counted runs, each named by segment and label ("Webster forks", "Plan reviewer"), with its cost, its share of the total cost and its cost per run.
- **Cost per run** follows: one row per run with the columns Discussion, Plan, Webster, Merge, Alignment, Orch, Other and Total, and an average row.
- **Segment tables** replace the role tables, in "All runs" and in each run's section.
  Each segment has a bold total row with its parts indented below it (`└ writer`, `└ reviewer`, `└ forks`, …), and both levels sort by cost, most expensive first.
  The columns are cost, share, time, time share and models.
- **`-detail`** adds the sessions, output, cache write, cache read, input and weight columns after the models column, and the total weight to the "All runs" total line.
- `+sub` shows nowhere: a fork role is its parent's part with " forks" appended, and the Orchestrator section's rows read `session` and `forks`.

## How roles map to segments

A role belongs to the segment whose key it equals or leads with, followed by `-`: `discussion`, `plan`, `webster`, `merge` and `alignment`.
The key role is labelled `writer`, except Webster's `Master` and Alignment's `session`.
The suffixes `-burler-review`, `-burler-fix`, `-bouncer-judge` and `-bouncer-seed` are labelled reviewer, fixer, judge and seed, and any other suffix is labelled by itself.
Webster's Master forks are labelled `forks` alone.
The webster recovery strand's `recovery` role is a Webster part.
Every other role, such as `describe`, `arbiter`, `conflict`, `friction`, a darn run's roles and `untitled`, is an Other part labelled by its own name.
The mapping lives in `segment.go` (`partOf`).

## Orch

`OrchCharge` now keeps the session and fork sides of each charge apart (`Session`, `Forks`, `Total()`), so the Orch segment shows `session` and `forks` parts per run.
The charging rule is unchanged.
In "All runs" the Orch segment also holds the orch usage charged to no run as an `unattributed` part.
That way the segment total and the Most expensive parts list carry the whole orch cost of the counted period, and the segment row names that cost averaged per run.
The per-run tables and the Cost per run table hold only each run's own charge, since the unattributed usage belongs to no run.

## Design decisions

- **Share is cost share.**
  A row's share is its cost over the table's total cost, the Orch segment's included.
  The old weight-based share went away with the weight column.
- **Time is a union.**
  A segment's time is the union of its roles' working intervals, so overlapping sessions count once.
  As before, "All runs" sums the per-run figures.
- **Sessions sit behind `-detail`** with the token columns, so the default tables hold only cost, share, time and models.
- **The parts list sits after the header.**
  It comes after the title and the generated-for lines, since those name the runs the list covers.

## Tests

`segment_test.go` pins the role-to-part mapping in a table, and in a two-run report it pins the parts list, the cost per run table and the order of segments and parts.
The existing report assertions in `count_test.go`, `orch_test.go` and the integration test `time_integration_test.go` now check the segment tables.
They also cover the `-detail` columns, the Orch segment's session, forks and unattributed parts, and the per-run average.
The usage text in `tokencount.sh`, `tokencount.cmd` and the package doc in `main.go` describe the new report.
