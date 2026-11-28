## Goal

```markdown
## Why

The bench of 2026-11-27 (`docs/research/weaver/bench-2026-11-27/report.md`) had only one plan where Weaver and Webster ran the same day,
so its quality and cost comparison is thin.
It is run again after Weaver's four named changes have landed (operator, 2026-11-28).

## Runs

Exactly six implementer runs and six reviews:

- The same three plans as the first bench: `board-viewer-server`, `launch-manifest-parity` and `gate-test-map-mode`, from the same base commits.
- Each plan once with Weaver on its default ladder and once with Webster on its default models, the two started together, the same day.
- One Burler round per result, as in the first bench: a `sonnet[medium]` reviewer with the code lens fan and the archived record and plan as the answer key.

Set up each run the way the first bench did, in a worktree pair of its own.

## Report

A new report beside the first one, with the same tables: wall time, gate time, fixer time, dollars, ladder and stops, verify, diff and review findings.
It checks that the four changes show: no attempt killed by the shuttle timeout, deviations in `summary.md`, and `gate_spans` on the Weaver rows.
It explains where Weaver's saving comes from, measured rather than guessed: cache reads per batch for Weaver against cache reads per fork for Webster.
It ends with a recommendation on making Weaver tapestry's default implementer.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-28): The orch makes the six bench pairs; this session runs them pair by pair after the batch-gate change lands

Decision: The orch created six pairs from this task worktree: wbr-b1 to wbr-b6, at /home/knatte/Code/loomyard-LYMHUB/wbr-bN with their -weft records sides. This session does everything else in them: commits each base tree, copies in the archived plan and decision record, seeds the run, moves it straight to the implementer row with a pause after that row, runs lym burler run, and collects the records. Assign b1/b2 board-viewer-server, b3/b4 launch-manifest-parity, b5/b6 gate-test-map-mode, odd Weaver and even Webster. Run one plan at a time, its Weaver and Webster runs started together, and the next plan once both have finished. Start the first plan only after batch-gate-tier1-own-packages has landed and its deploy has run, so every run uses one lym build; ask the orch if it has not landed when you are ready. Record the load average at each start in the report. Do not push the bench branches and do not remove the pairs: say when they are done and the orch removes them.

Rationale: A task session cannot run lym fabric add, so the orch makes the pairs from the task worktree and the task agent runs the bench (operator, 2026-11-16). One lym build across the bench avoids the first bench's mid-bench rebuild limit, and one plan at a time keeps the machine's load from swamping the comparison while each plan's two implementers still share it.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
