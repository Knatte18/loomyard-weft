---
round: 2
focus:
  - "In decision-record.md D1 and D2, settle whether the signal that routes an escalation to the settling fork excludes the arbiter's verb-concluded hand-ons: internal/loomcli/arbiter.go `handOn` writes an envelope with verdict hand-on and a non-empty cause (timeout, died, missing, unparseable, precondition) for those cases, and internal/loomengine/arbiter.go says cause is empty only for the agent's own verdict. Check that D1's signal and D2's fault list cannot both claim the same escalation, and that the path handed to the fork exists in every case D1 covers."
  - "In decision-record.md D1, settle whether the Decision states what the settling fork can let through and what bounds it, given that it may record decisions, resume the run and run the brief's decide options such as `lyx loom circling accept`, and that the arbiter stencil loom-template-arbiter.md hands on when a call needs an unrecorded operator decision, widens scope, or leaves doubt. Check the brief's settle condition mirrors that bound."
  - "In decision-record.md D3, settle whether a peer escalation is classified under D1's hand-on rule and D2's fault rule before it reaches the joint fork, whether the joint fork is bound like D1's fork, and what a late-peer fresh fork reads while the first fork is still open."
  - "In decision-record.md D7 with its Open risks and Acceptance criteria, settle what happens to the moved issue-673 workaround when the parent's CLAUDE.md loses it between this run's Plan-Write and its landing, and whether some Decision or action owns that case."
---

Entries name where to look; the reviewer judges each against the current record.
