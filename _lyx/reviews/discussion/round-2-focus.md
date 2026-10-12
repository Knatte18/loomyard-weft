---
round: 2
focus:
  - "decision-record.md D1: the state cell renders the batten status's literal state, and `internal/shedengine/status.go` declares exactly running, paused, done, blocked, failed and awaiting. Settle whether every state a covered run can be in has a stated rule in D1 and in the acceptance criteria, and whether any state name used there is not one of those literals."
  - "decision-record.md D2: `internal/shuttleengine/agentwait.go` counts a run as waiting when its held-turn wait marker is live or its session reads asking or idle-stalled. Settle whether D2's held set for the advisor predicate states which of those readings hold the chair's notice, and whether the Decision or its Rationale says why any omitted reading still defers it."
  - "decision-record.md D3: the text reaches tmux as stdin bytes of `load-buffer -`, and both shuttle's sends and reed's name repair in `internal/reedengine/namerepair.go` reach the paste path. Settle whether the stated bound for the removed escape and dash guard is true for every caller and names what the change can let through."
  - "decision-record.md D4: global flags typed before the shortcut name reach the expansion. Settle whether the Decision states which argv of a multi-command expansion each forwarded word reaches, and whether the acceptance criteria pin that rule."
---

Focus targets the Decisions that carry a stated bound or a state or reading set checked against code.
Relocation and exclusion remain legitimate findings; the `Do not flag` list and the writer/reviewer symmetry rule apply as in round 1.
