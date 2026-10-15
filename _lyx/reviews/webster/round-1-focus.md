---
round: 1
focus:
  - "Webster adapter reroute (card 4): check that a stuck outcome with a new conflict reroutes and a repeated conflict halts, and that 'new' versus 'repeated' is decided from the conflict record the way cards 2 and 3 define it."
  - "Conflict record lifecycle (card 2): check that begin-batch writes the record, removes it on a clean outcome, that Run removes it on entry, and that the record is never tracked in git."
  - "Dispatch refusal (card 1): check that no path still reports `plan_drifted` for the blocking refusal, and that both causes (`plan_defect`, `tree_moved`) reach the `plan_conflict` envelope."
  - "Recipe edge (card 3): check that `on_reroute` on the Webster row targets Plan-Burler-Review and that the cap clause is row-neutral in every place it is stated."
  - "Config editor (cards 5, 6): check that `EditPath` restores the previous bytes on an empty or whitespace-only result on every editor path, and that the no-terminal refusal covers both bare `lyx config` and `lyx config menu`."
  - "Test economy: check new tests against existing coverage, naming the covering test for any duplicate."
---

Seed focus from the plan's card index; the per-card mechanical checks and comment conventions apply as the rubric states.
