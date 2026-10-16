---
round: 1
focus:
  - "D5 (revive): does the Decision state what the new send path can skip or let through, and what bounds it? Check the race with a pause, the stale-state window between the state read and the send, and the orch notice dedup."
  - "D6 (#606): does the deadline extension state its bound for a caller context that carries a deadline of its own and for several requests under one deadline? Is the premise that dropping the http client `Timeout` lets the transport own both deadlines sound, and does the Open risk cover a client built elsewhere?"
  - "D8 and D9: do the re-exec gap (second watcher, lost `refresh`/`distill` request) and the exec that skips the root's post-run state what they let through and what bounds it? Does the shortcut-guard bound in D9 cover every shortcut that can end in `reed attach`?"
  - "D3 and D7: `status` is open to any caller, agents included, and lists sessions and process argv. Is the read scope stated and bounded, and is the stale-binary matching rule (deleted exe by base name, cwd under the hub) unambiguous enough that the plan writer cannot read it two ways?"
  - "Scope Out and Constraints: does each exclusion or constraint carry a requirement the plan writer needs, or is it deliberation that belongs in `support-log.md`? Apply the completeness-before-leanness test before any relocation finding, since `Plan-Write` never reads the support log."
  - "Consistency between the Scope bullets, D1-D9 and Acceptance criteria: does every Scope item have a Decision, and does every bound or behavior a Decision states have a matching criterion?"
---

Round 1 seed.
The writer's stencil forbids exact signatures, `file:line` citations and exhaustive pattern research, so none of those is a focus.
Missing rejected alternatives, a missing Notes subsection and call-site enumeration are excluded by the rubric and are not entries.
