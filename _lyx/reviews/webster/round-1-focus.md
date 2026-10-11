---
round: 1
focus:
  - "Convergence guard (card 5): does the replay path skip the guard for a round with an escalation record or a circling decision file, and does the guard archive the judgment and re-judge within the bounce budget rather than loop or spend extra budget?"
  - "Convergence guard (card 5): does the ledger-key comparison refuse exactly a judgment that drops a key of the previous round's ledger, and does CONVERGED over an open BLOCKING finding get refused on both the live and replay paths?"
  - "Row-keyed bouncer models (cards 3, 4): does a Bouncer row with no `Env.RowBouncerModels` entry fail at construction, and are the seed and judge pairs threaded per segment from run start with the old row model keys fully retired, including their callers, docs and templates?"
  - "Empty-key fallback (cards 1, 2, 4): with every new key empty, does each role resolve to the model it ran on before, including `rework` falling back to `plan`, and do the load-time grammar checks accept empty values?"
  - "Usage record (cards 6, 7): does the `*halfUsage` change keep a Burler-written record's on-disk form unchanged, does a record without a review half read as missing in every reader, does merge-on-write keep earlier seed and judge entries, and does the Bouncer record every seed and judge pass whose result reached it?"
  - "Per-card mechanical checks for every Rename or Delete group in the cards, run against each group's own targets, and test economy for the new tests against the existing ones in the same packages."
---

Focus is set from the plan's card index and shared decisions alone; no diff-specific finding is pre-stated.
