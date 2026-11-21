# Burler fixer step re-runs on a round an earlier fixer finished, with no reason in the prompt

`burler-fixer-rerun-no-reason` · Note · bug, burler

Imported from [issue #176](https://github.com/Knatte18/loomyard/issues/176).

What happened: the Plan-Burler-Fix step for round 1 started a fixer session after an earlier fixer session had already fixed every finding and run its self-check.
The earlier fixer's report sat under a timestamped name (`round-1-fixer-report-<UTC timestamp>.md`) rather than the expected `round-1-fixer-report.md`.
The prompt gave no reason for the second run, so the fixer had to read every card to tell "re-run because the report was misnamed or rejected" from "re-run for new work".
It wrote the expected report from the earlier dispositions and spawned no forks.

Why abnormal: the step's report-name expectation and the name the first fixer wrote disagree, so a finished round looks unfinished and costs a full fixer session.
The prompt also never states why a step runs again.

What to do: find why the first fixer's report landed under the timestamped name and was not accepted as the round's report (archiving on write, or a name mismatch).
Either accept the archived report, or state the re-run reason (misnamed or rejected report versus new work) in the fixer prompt.

Provenance: task single-repo-namespace-layout; notes burler-plan-r1.md.