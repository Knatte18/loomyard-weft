---
round: 2
focus:
  - "internal/alignment/session.go (`Session.Begin`, `Poll`): settle whether the alignment attempt's timeout counts only awake, unpaused time from the start's return, as the task's decisions D4, D5 and D9 require, including a start held at entry by a pause and a Poll after a covered interval or a detected sleep."
  - "internal/loomcli/landingdeps.go (`landingAlignmentDeps`) and internal/loomcli/wiring.go (`newParentReviewAligner`): settle whether both alignment users receive the pause geometry and slug the session needs, so neither the PR-Align nor the parent-review aligner falls back to a wall-clock deadline."
  - "internal/landingshed/pralign.go (`wait`) with `Deps.Lock.PauseRequested` in internal/landingshed/deps.go and `parentLockDeps` in internal/loomcli/landingdeps.go: settle whether a `--after` naming PR-Align ending the alignment wait mid-row agrees with the task's D8 bound, the refusal-spec row for a paused parent lock wait, and internal/landingshed/doc.go."
---

Focus targets the merged interaction of the parent's alignment session with the task's pause and sleep model.
