# Bench a Sonnet darn writer with an Opus advisor on three darn tasks

Adds the report of a bench that reran three landed darn tasks with a Sonnet writer consulting an Opus advisor, set through `lym shuttle run --advisor`.
The change is one new document, `docs/research/loom/advisor-model-bench-2026-11-12/report.md`; no code changes.

The report states the advisor prompt in full at its top, then compares each rerun with the task's recorded Opus run on cost, time, diff size, review findings and advisor consults.
On this sample the Sonnet writer with the advisor cost 10% to 35% less than the recorded Opus writer, the saving shrinking as the task grows.
It drew a BLOCKING or MEDIUM review finding on every task, and it did not match the Opus writer's first-pass quality.

## Design decisions

- Exactly three runs, one per task, and no prompt iteration, as the task caps.
- The reruns ran in local clones at the tasks' base commits, outside the hub, so no test, verify gate or cold fixer ran for them; the report says so and does not count tests in the comparison.
- One Opus reviewer stands in for the Code-Review perch, which needs a pair.
  The report adds the same reviewer on each landed Opus diff as a control, because the recorded review used different reviewers.
- The report records no recommendation on a lym config key for the advisor; it names what a deciding bench would need.
