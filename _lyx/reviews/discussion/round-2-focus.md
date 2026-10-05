---
round: 2
focus:
  - "Decision D6 in decision-record.md deletes a per-worktree _lyx/config/board.yaml under reconcile --apply once the hub file exists. Settle whether the Decision, or its Bound, says what happens when that copy holds keys, types or labels the hub file lacks, so a vocabulary edit made only in the per-worktree copy is neither dropped without a signal nor left unstated."
  - "Decision D4 seeds the hub board.yaml from the prime worktree's copy, falling back to the embedded template. Settle whether the record states what the seed does when no prime worktree resolves or the prime has no board.yaml, and whether that case degrades silently to the template, which the support log's rejected LoadOrTemplate alternative says drops custom types and labels."
  - "Decisions D4 and D5 and the Acceptance criteria name a commit of the hub board.yaml in _board. Settle whether every write path that changes that file (the reconcile seed and the lyx config board edit) has a stated commit route, lock, push-failure outcome, and validation before commit, so the plan writer invents none of them."
---

Targets the places where a Decision adds a write path or removes a guard, per the rubric's attack-surface question.
Nothing here asks for enumerated call sites or rejected alternatives in `decision-record.md`.
