---
round: 1
focus:
  - "D7 and its Bound: does the sweep's deletion rule (target-path match, dangling targets, foreign entries kept, the retired workspace file removed as a link) state what it can delete and what bounds it, and do the acceptance criteria match that rule?"
  - "D4 and D3: a regeneration failure never fails or rolls back a verb. Is the stale-list bound true for every listed verb, including Prune with apply and the Add rollback path, and do D3's 'writes nothing' and D4's 'regenerates on every exit past pre-flight' agree?"
  - "D5: the lock is named only as a leaf lock in the board's never-tracked state dir. Does the Decision say enough for the plan writer to place it and to know what the bounded wait does on timeout, or is a requirement left implicit?"
  - "D2 and D6 against the Scope list: does every Scope item have a Decision that delivers it, and does every Decision stay inside Scope (the `ideengine` change, the reserved-name set, the docs)?"
  - "Acceptance criteria against Decisions: does each criterion test a behavior a Decision states, and does each Decision with a bound or failure mode have a criterion?"
  - "Premises that read code claims: the D2 listing premises (worktree listing skips unresolvable entries, weft worktrees never listed, `--local` pairs and pairs forked from pairs are code worktrees of the hub) and the D3 no-import-cycle claim. Is each stated as a decision the plan writer can rely on, or as an unverified assumption?"
  - "Relocation check: does any Decision or Rationale carry deliberation narrative that belongs in support-log.md, after first confirming it holds no requirement the plan writer needs?"
  - "PATTERN changes against Constraints: do the two listed entries cover every invariant D3, D6 and D7 add or loosen, and do the Constraints contradict any Decision?"
---

Seed focus only; the reviewer still reads both files in full.
No Do-not-flag item is promoted: no entry asks for a Notes subsection, rejected alternatives in the record, or call-site enumeration.
