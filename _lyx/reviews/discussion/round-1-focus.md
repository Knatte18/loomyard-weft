---
round: 1
focus:
  - "D1 and D4: does every load site that stacks the machine layer (worktree loads with Board and Anchor) and every site that must not (Board-only hub-file checks, standalone loads) follow from the stated rule, with no site left ambiguous, including the machine write's own CheckLoad and the editor path?"
  - "D2, D3 and D5: is the retired-key handling consistent across the machine layer's admit rule, 'every load drops retired keys', and the D3 failure message, so a stale or refused key in the machine file has one stated outcome?"
  - "D3 and D4: for each new bound-affecting edge (the machine override lowering or raising `slots` and `go_parallel`, the `--layer machine` verb, removing the file as a way forward), does the Decision state what it can skip or let through and what bounds it?"
  - "Acceptance criteria against Decisions: does each Decision's observable behavior have a criterion, and does no criterion assert behavior no Decision states?"
  - "Open risks and Constraints: does any Constraint or PATTERN change contradict a Decision, or carry deliberation that belongs in support-log.md, applying the completeness-before-leanness test before any relocation finding?"
---

Seed focus for round 1.
No finding is pre-stated; each entry names where to look and the question to settle.
