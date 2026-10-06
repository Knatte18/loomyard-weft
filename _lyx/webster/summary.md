# Skills in one turn: stuck at batch 04

Batches 01 and 02 are done and committed; the run stopped at batch 04, and batch 03 never began.

## Done

- 01-skill-load-transcript (`c42c70995c958f5c74699974a5664db04d17e6f3`): added `SkillLoadReport` and Claude's one-message skill load and transcript classification.
  `record-batch` reported no deviations; its warnings were informational scope notes only.
- 02-launch-skills-one-turn (`f3cd9b136a5f72c263db621f9dd929f0acd3e198`): shuttle's launch loads the whole skill list in one turn with at most one retry, and the per-list `Runner` primitives were added.
  `record-batch` reported no deviations; its warnings were informational scope notes only.

## Stuck

- 04-drop-per-skill-load: the fork reported failure and made no commit.
  Its stated reason is that card 4 deletes `Runner.LoadSkill` and `Runner.SkillUnknown`, which the orch code (`internal/orchcli`, `internal/orchengine`) still calls until card 3 lands.
  The batch order derived by webster runs 04 before 03, so the delete targets still resolve.
- `record-batch 04` refused with `batch_failed` and archived the report.
  `recover-batch 04` refused with the same `batch_failed` and the same `delete-not-done` findings, so recovery is exhausted.

## Way forward

Card 4 needs to run after card 3.
Edit the plan so card 3 precedes card 4 (or declare card 4's dependency on card 3), run `lyx webster rebaseline --card NN` for each edited card, and resume with `lyx webster run`.
