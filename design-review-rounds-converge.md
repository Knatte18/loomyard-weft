# Review rounds converge: fixer rebuttals, incremental re-review, stalemate on no tree change

`review-rounds-converge` · Task · feature, burler, shed, stencil · starting · Open-Window

## Seen (orch, 2026-11-17)

Burler's review rounds circle more than they need to, in three ways.

- A fixer can fix a finding, leave it undone or defer it to the parent, but its argument against a finding never reaches the next round,
  so the reviewer can raise the same finding again on the same code.
- Each round after the first reviews the whole diff again, so it can raise new findings on code it passed before.
- A run's escalation cause `circling` (`internal/loomcli/circling.go`) is an LLM's judgment,
  though a round whose fix changes nothing in the warp tree is a stalemate a Go check can see.

Sources, from the survey F17 (`orchestrator-survey-ideas`):
ralphex's external review, where the agent fixes each finding or argues against it and the rebuttal goes into the next round,
and its `--review-patience=N`, which stops after N rounds with no tree change;
Traycer's re-verify, which checks only earlier findings and marks each resolved or outdated;
Fusion, which forbids new non-blocking findings on a re-review.

## Idea

- **Rebuttal.** The fixer may answer a finding with a rebuttal: why the code stands, with file and line.
  The next round's reviewer gets each rebuttal beside its finding and either withdraws the finding or answers the rebuttal.
  It applies to every reviewer engine, Codex included.
- **Incremental re-review.** A later round checks the earlier round's findings against the fixes and marks each resolved, still open or outdated.
  It raises a new finding only on code the fixes changed, and only at BLOCKING severity.
- **Stalemate on no tree change.** The Bouncer counts rounds whose fixer left the warp tree unchanged,
  and after a set number in a row halts the segment with a named stalemate cause before the bounce budget runs out.
