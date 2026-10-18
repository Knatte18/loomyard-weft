# tokencount reports cost per segment, most expensive first, each split into its parts

`tokencount-segment-report` · Task · enhancement, report · running · PR-Gate

## Goal

The tokencount report must show at a glance what costs most in a run, so the operator can see where saving is worth trying.
Today it lists one flat row per role and per `+sub` role, so the cost of one segment means adding up four to six rows by hand.
The `+sub` name also reads as "the role plus its forks", while the row holds only the forks.

## Scope

All changes are in `tools/tokencount`.

1. A "Most expensive parts" list opens the report, before anything else: the ten most expensive leaf parts across every segment, each with its cost, its share of the total and its average cost per run.
   A part is one sub-row of the segment tree below, such as "Webster forks" or "Plan reviewer".
2. A summary table follows, with one row per run and the columns Discussion, Plan, Webster, Merge, Alignment, Orch, Other and Total, in dollars, with an average row.
3. The roles are shown as a tree: each segment is one total row (Discussion, Plan, Webster, Merge, Alignment, Orch, Other), with indented sub-rows under it that sum to the total (writer, writer forks, reviewer, fixer, judge and the like; Webster's are Master and forks; Orch's are the orch session and its forks).
   Alignment is its own segment.
   Orch is the orch usage the tool already charges to each run by time, and its segment also shows the average orch cost per run.
4. Cost and share are the first two number columns of every table, followed by time and models.
   Segments are sorted by cost, most expensive first, and so are the sub-rows within each segment.
5. `+sub` is shown as `forks` everywhere.
6. The token columns (output, cache write, cache read, input, weight) move behind a `-detail` flag.
7. The tests in `tools/tokencount` and the usage text in `tokencount.sh` and `tokencount.cmd` follow the change.

## Out of scope

How sessions are counted, priced or charged to runs stays as it is.
