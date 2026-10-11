---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md`, the Decision that routes an interim fork notice to a turn end with no tool call: does it state what it lets through and what bounds it, for a fork that never sends a final report and for a final notice whose text is misread as interim? Check against `contracts/stencils/webster/webster-template-master.md` and shuttle's `expiredTurnEnd` in `internal/shuttleengine/wait.go` whether a fork that has sent a finished notice still counts as outstanding."
  - "In `_lyx/discussion/decision-record.md`, the Decisions and Open risks on removing interim notices and allowing foreground gate calls of up to an hour: is the premise settled, whether an outstanding fork or a fork's foreground call counts as activity for `step_idle_timeout_min` in `internal/loomengine/template.yaml` and `notice_quiet_min` in `contracts/recipes/batten-recipe.yaml`, and if not, does a Decision carry the remedy as a requirement rather than leaving the plan writer to investigate?"
  - "In `_lyx/discussion/decision-record.md`, the Decision on the verify-fix fixer fork and the Master template's gate-failure fixer step: is the fixer fork's completion-notice path guarded against interim notices the same way as the batch loop, and does its body carry the same background ban and quick-check rule?"
---

Focus entries restate their sites so the reviewer needs no earlier round.
