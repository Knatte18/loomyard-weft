# Decision record — tokencount reports cost per segment

## Goal

The tokencount report shows at a glance what costs most in a run, so the operator sees where saving is worth trying.
Today it lists one flat row per role and per `+sub` role, so one segment's cost means adding up several rows by hand.

## Scope

All changes are in `tools/tokencount`.

- A "Most expensive parts" list opens the report body: the ten most expensive leaf parts across every segment, each with its cost, its share of the total and its average cost per run.
- A "Cost per run" summary table follows: one row per run, the columns Discussion, Plan, Webster, Merge, Alignment, Orch, Other and Total in dollars, and an average row.
- The role tables, "All runs" and each run's, become segment trees: one total row per segment, with indented sub-rows that sum to it.
- Cost and share are the first two number columns of every segment table, followed by time and models; segments and sub-rows sort by cost, most expensive first.
- `+sub` is shown as `forks` everywhere in the report.
- The token columns (sessions, output, cache write, cache read, input, weight) move behind a `-detail` flag.
- The tests in `tools/tokencount` and the usage text in `tokencount.sh` and `tokencount.cmd` follow the change.

Out of scope: how sessions are counted, priced or charged to runs.

## Decisions

### Segment by role prefix

Decision: a role belongs to the segment whose key it equals or starts with followed by `-`: `discussion`, `plan`, `webster`, `merge`, `alignment`.
The `recovery` role, webster's cold recovery strand, belongs to Webster.
Every other role, such as `describe`, `arbiter`, `conflict`, `friction`, a darn run's roles and `untitled`, belongs to Other.
Orch is its own segment, built from the orch charge the tool already makes per run.
Rationale: the loom recipe's strand roles all lead with their segment's prefix (`plan-burler-review`, `merge-bouncer-judge`), so the prefix is the segment without a second table to keep in step.

### Sub-row labels

Decision: the role equal to the segment key is the writer, labelled `writer`, except Webster's, labelled `Master`, and Alignment's, labelled `session`.
The suffixes `-burler-review`, `-burler-fix`, `-bouncer-judge` and `-bouncer-seed` are labelled `reviewer`, `fixer`, `judge` and `seed`;
any other suffix is labelled by itself without the leading dash.
A `+sub` role is labelled `<parent label> forks`, except Webster's Master forks, labelled `forks`.
In Other each role is labelled by its own name.
Orch's sub-rows are `session` and `forks`.
A part's name in the "Most expensive parts" list is the segment name and the label, as in "Webster forks" or "Plan reviewer".
Rationale: these are the names the board entry uses.

### Share is cost share

Decision: the share column is a row's cost as a percentage of its table's total cost, the orch segment's included.
The weight-based share is gone with weight behind `-detail`.
Rationale: the report now ranks by cost, so its share must be of the same figure.

### Orch charge split into session and forks

Decision: each run's orch charge keeps its total and gains the part charged from orch sub-agents, so the Orch segment shows its `session` and `forks` sub-rows per run.
The charging rule is unchanged.
The "All runs" table's Orch segment row names the average orch cost per run.
Orch rows carry no time and no models, as before.

### Segment time is a union

Decision: a segment's time is the union of its roles' working intervals clipped to the span; in "All runs" it is the sum of the per-run figures, the way roles already sum.
Rationale: summing sub-row times would double-count overlapping sessions.

### The -detail flag

Decision: `-detail` adds the sessions, output, cache write, cache read, input and weight columns after the models column of every segment table, and the total weight to the "All runs" total line.
Without it the tables hold the segment or part, cost, share, time, time share and models.
It changes no figure and no other section.

## Constraints

- PATTERN-test-economy and PATTERN-test-speed: extend the existing tests in `tools/tokencount` rather than add parallel ones; tier-1 fakes only.
- PATTERN-comment-line-breaks: one sentence per line in Go comments.
- The package doc in `tools/tokencount/main.go` describes the report and changes with it.

## Auto-mode assumptions

- "Before anything else" means first in the report body, after the title and the generated-for header lines.
- The Webster forks, Orchestrator, Producers and calibration sections keep their content, with `+sub` shown as forks.
- Sessions count among the token columns moved behind `-detail`, since the default tables hold cost, share, time and models only.

## Open risks

- A future role whose name does not lead with its segment's prefix lands in Other until the prefix table learns it.

## Acceptance criteria

- The report body opens with "Most expensive parts", ten leaf parts at most, each with cost, share of the grand total and cost per run, sorted by cost.
- A "Cost per run" table follows with one row per run, the seven segment columns, a Total column and an average row.
- Each segment table has segment total rows with sub-rows summing to them, sorted by cost within each level, and an idle row.
- An Orch segment appears with `session` and `forks` sub-rows when the orch was counted, its "All runs" row naming the average orch cost per run.
- `+sub` appears nowhere in the report; `forks` replaces it.
- The token columns appear only under `-detail`.
- The existing tests and the usage text in `tokencount.sh` and `tokencount.cmd` follow the change.
