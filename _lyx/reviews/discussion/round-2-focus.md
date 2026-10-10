---
round: 2
focus:
  - "D1 routes a Publish verify that fails, times out or cannot start to the fixer round: check, against `verifyGate.recordFailure` in `internal/landingshed/verifygate.go`, whether a failure record is written in each of the three cases, and whether the Decision, the Goal, the first Acceptance criterion and the `Publish: verify failed` spec row all agree with what the record carries."
  - "D3 lets `lyx loom start` and resume through the unfinished-merge refusal for a run halted at Publish or Finalize: check `refuseOverUnfinishedMerge` in `internal/loomcli/start.go` for a run still `running` at Publish or Finalize whose driver died mid conflict session, and whether the Decision names its way forward."
  - "D6 refuses a `--by parent` or `--by operator` call from a strand whose name carries a slug: check the check against a standalone run, whose agents are named `s<hash>:<role>` with an empty `NameSlug` and which has no parent, and whether the Bound and the new plan-stencil text are correct there."
  - "D6 puts a session name into the refusal way forward and the shipped plan stencil: check whether the name is a literal or resolved per hub and per task, given that the shortname is read per hub and stencils are seeded into every hub."
  - "Every Decision that adds an abort, a route or a guard bypass states what it can now discard or let through and what bounds it: check D2's abort against a parked merge of the parent branch that someone started by hand with `lyx fabric merge-in`."
---

Focus targets the design-class questions that gate a round.
Each entry names the code to read and the question to settle; none pre-states an answer.
