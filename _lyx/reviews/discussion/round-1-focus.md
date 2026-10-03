---
round: 1
focus:
  - "D3 steps 1 and 4 (pass without verifying on an absent or non-done outcome.yaml, and on a missing `## verify:` section): settle whether each pass-through states what it lets through and what bounds it, in particular what an absent outcome.yaml at a Done arrival leads to after the gate passes."
  - "D7 and D15: the verified-tree record under `.lyx` lets every verify site skip. Settle whether the Decision bounds who can write that record (Merriam, the fixer fork, a Burler round session, the conflict session, or `lyx webster verify` run by an agent), or whether the skip's only stated bound is the tree match."
  - "D8: an exhausted Webster-Burler `verify` gate routes Stuck to Webster-Bouncer. Settle whether D8 itself states what that edge lets through and what stops a red tree, or whether that bound lives only in Open risks, where Plan-Write may not treat it as a requirement."
  - "D1 against the Open risks entry that says the fork, not Go, runs the card gate: settle whether D1 states that the per-card gate is prompt-rendered and not enforced, and what bounds a fork that skips it."
  - "D5's commit check: settle whether it is consistent with the other writes that follow a gate failure (Merriam's outcome.yaml and summary.md rewrites, any mid-run parent merge, which the non-merge rule would refuse), and whether its Terminal outcome is reachable by legitimate work."
  - "D3 step 5 and the Constraints entry on in-memory attempt counts: settle whether the flaky-rerun pass and the attempt reset after an attach are stated bounds on the gate the plan writer can implement against, or need to move into the Decision."
  - "Acceptance criteria against Decisions: settle whether each Decision with observable behavior has a criterion, notably D2's Plan-Write grep rule and D6's blocking `recover-batch` (the stencil-absence criterion covers only the removed loop)."
  - "Relocation and exclusion: settle whether incident narrative in Rationale lines (D4's card 07/card 09 account, D7's board-entry reference) belongs in support-log.md, applying the completeness-before-leanness test first, and whether the Notes for the plan writer's code-level enumerations exceed the stencil's Step 2 exploration bound. Absence or incompleteness of that optional section is not a finding."
---

The focus weights the attack-surface question, since this task adds several skip paths (D3's pass-throughs, D7's record-based skip, D8's Stuck edge, D10's widened staging) and moves a gate (D1) from Go into a prompt.
D5, D6 and D10 already state an explicit bound; they appear here only where a consistency question remains.
Per the writer's stencil, missing rejected alternatives in decision-record.md, missing call-site enumeration, and a missing or partial Notes section are excluded.
