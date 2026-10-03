# PR-Gate review: agent-self-reflect (PR 355)

Verdict: reject.
The decision record's D1–D7 are implemented, and `go test ./...` and `go test -tags integration ./...` both pass.
Two items in the task's board goal are missing.
The operator approved both, and the hub relayed them to Plan-Write after Discussion.
Plan-Write's first plan (weft `d26e6da`) carried them as cards 7 and 8.
Plan review then dropped them: the final `00-overview.md` "Decision: the board's post-Discussion additions are not carried" says they need the decision record amended first.
That is wrong for this task, because the additions come from the operator, not from drift, and the board entry records them under "From `convergence-plan`: Friction on halt".

## F1 — Tier 1 anomaly filing does not run under `step` (blocking)

Board goal: "Tier 1 anomaly filing (`detectAndFileAnomalies`) also runs under `step`, since the no-gate decision covers every filing path."
Today only `loomPostRun` calls `detectAndFileAnomalies`, so a driven run never files escalation-to-human, bounce-budget-exhausted, producer-hard-failure, recurring-finding or crash-resume anomalies.
Every orch-driven run uses `step`, so the goal "every run's friction reaches GitHub with no operator in the loop" is not met for these.

Fix: call `detectAndFileAnomalies` from the new `AfterStep` hook, ahead of the D2 halt note and reflection.
Keep its existing skips: knob off, `ErrShedBusy`, cancelled context.
Take the entry observation after the busy probe, so a completed step does not read as a crash.
State which path owns which filing so one halt is not filed twice.
Tier 1 files the halt event, and when the selfreport knob is on, the halt note says so and the reflection files only the lyx problems behind it.
Pin it with tests through `loomCLI.fileIssue`.

## F2 — A halt whose filing failed is lost once the run resumes (blocking)

Board goal (moved from `convergence-plan`): "Check first whether `internal/loomengine/anomaly.go` loses a halt that is later resumed."
Plan-Write's own analysis found that it does, in both verbs.
A failed `FileIssue` leaves the title "to retry next run", but detection reads only the final status, so a halt that was later resumed is never filed.

Fix: keep failed anomalies as pending entries in the filed-title marker, and retry them on every later detection pass.
Pin it with a test: filing fails, the run is resumed, and a later pass files it.

## Non-blocking

- No new file was added under `manifest/designs/`, and the branch contains `main` through `4d4d1bafc`.
- Nothing files through `gh`, and no approval gate remains: ly-drive files nothing, and the reflection stencil files through `lyx selfreport create` only.
