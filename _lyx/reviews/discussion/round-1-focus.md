---
round: 1
focus:
  - "Residue of the hub review's D5 replacement: check every section that mentions delivery against the SendMessage design. Does the Question ledger's Q10 (hand-started hub gets no push notice), Interview turn 4 (reed SendText by guid), the Acceptance criteria's 'hub-wide name resolution' unit test, and the Scope's 'orch start stencil' versus D12's 'start and resume stencils' still agree with the revised D4/D5/D12?"
  - "D5 and the Agent Name Invariant constraint: which address does the delivery use, `$LYX_PARENT` from the writer's environment or the full name recorded in the seed, and does the record say how the two relate when they differ or one is missing?"
  - "D4/D5 re-send loop: does a `lyx loom review delivered --failed` record stop or continue the once-a-minute re-send, and what bounds the number of notices the parent receives when a writer sends the SendMessage but never stamps delivery (up to one per minute for the whole wait bound)?"
  - "D6 attack surface: the verbs check no caller identity, and the live writer already runs `lyx loom review delivered` in its own worktree. Does D6 state what an unchecked `approve`, `reject`, `delivered` or `notify` lets through (for example a writer approving its own discussion, or stamping delivery without sending) and what bounds it?"
  - "D2/D5 interaction with arrivals: the delivery prompt makes the writer take a turn and end it, which is a Done arrival. Does the record settle how that arrival interacts with the gate list (re-running `discussion`, counts, the pending entry), and how a delivery prompt or reject re-prompt is sent while the writer is mid-turn?"
  - "Decision-record leanness: D5's dated hub-review rationale, the '(revised by the hub review)' auto-mode assumption, and similar process history. Apply the completeness-before-leanness test: which parts carry a requirement the plan writer needs, and which are deliberation narrative that belongs in the support log?"
  - "D8 and the writer stencil's write boundary: the writer is told to write a parent-review entry under the support log's `## Review rounds`, which the writer stencil reserves for the Discussion-review gate, and to run a `lyx loom review` verb that writes the request directory outside its two files. Does the record state the stencil change that permits both?"
---

The hub review replaced D5 in place and listed the sections that follow from it, so round 1 starts where a partial in-place edit leaves contradictions.
The other entries cover the new verbs and the re-send loop under the rubric's attack-surface question, and the writer stencil's own boundaries under the symmetry note.
