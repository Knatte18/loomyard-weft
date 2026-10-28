# Decision record: review-setup-audit

## Goal

Write `docs/research/review-setup-audit-2026-10-28/report.md`: one audit, from landed-run records only, of what each part of the review setup is worth.
It answers four questions: rounds per perch, per-lens worth, finding classes, and fix-delta review.
Each question ends in a recommendation marked as a recommendation, with the sample size beside every number.

## Scope

- In: loom runs landed since `79bab47a0` whose weft archive tag holds `*.lens-<lens>.md` files, read from the archive tags of `loomyard-weft`.
- In: `lym report convergence`, `lym report timing`, `tools/tokencount/tokencount.sh`, and the rounds' review files.
- In: committing the helper scripts and extracted tables that the report's numbers derive from, beside the report.
- Out: runs from OSL-1046, any bench, replay or grading agent, any change to `loom.yaml`, stencils, profiles or Go code.
- Out: a change to this task's board entry.

## Decisions

### Read the sample from archive tags, not from the live worktrees

Rationale: the task names the weft archive tags as the source, and live `_lym/` trees are torn down on landing.

### The writer judges real-or-noise and says so

Rationale: the task forbids grading agents.
A finding counts as real when the fixer changed code, plan or record for it, or the next round re-raised it unfixed; a finding the fixer declined or the bouncer dismissed counts as noise.
The report states this rule and that it is the writer's judgement.

### Recommendations are not decisions

Rationale: the profile ladder and lens sets belong to the operator.
The report changes no configuration.

### Numbers derive from committed scripts

Rationale: a reader can rerun the extraction against the tags.
The report cites the script and table beside each figure.

## Constraints

- `PATTERN.md` binds: no Go, config or stencil edit is made.
- Markdown follows semantic line breaks.
- No model or agent run beyond this session.

## Auto-mode assumptions

- Runs whose archive tag lacks lens files cannot be attributed per lens and are counted only in the round-level tallies where they hold round records.
- A lens file's findings are counted by the bullets or headed items its report format uses; where a report is prose, the writer counts distinct gating items by hand.

## Open risks

- Token and minute figures exist only for rounds whose `*-usage.yaml` or timing record survived in the tag.
- Real-or-noise is a single-judge call, so small counts per lens are indicative only.

## Acceptance criteria

- The report answers questions 1 to 4, each with a sample size beside every number and a marked recommendation.
- Every figure derives from a committed script or table.
- The report names its judging rule and its exclusions.
