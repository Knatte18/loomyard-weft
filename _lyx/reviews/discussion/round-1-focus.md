---
round: 1
focus:
  - "Decisions 2, 3 and 7 on `merge`: settle whether the upserted entry's `status` change inside a merge is judged by the same rule as Decision 3's single upsert, so no merge path edits and unlocks in one write."
  - "Decision 3's carve-out and Decision 5's refusal, which names `set-status` as the unlock: settle whether anything bounds who or what may clear a run's status mid-run, or whether the stated bound (a separate visible write) is the whole contract."
  - "Decision 1 on hand-set statuses: settle whether any status value a non-run flow writes today would lock an entry by accident, and whether the Auto-mode assumption covers it."
  - "Decision 8 and Decision 4 together: settle whether the lock window between the claim write and loom's first transition has a stated bound, including when the claim skips a `done` or absent entry."
  - "Decision 6 against the Acceptance criteria: settle whether the refusal-before-GitHub ordering is stated as a requirement the plan writer can act on, and whether each Decision's behavior has a matching acceptance criterion."
  - "Decisions and Acceptance criteria for duplicated or contradictory statements, and for deliberation narrative that belongs in `support-log.md` once any requirement it carries sits in a Decision or Constraint."
---

Seeded from a read of both artifacts and the discussion stencil.
Nothing here asks for exact signatures, call-site lists, rejected alternatives or a plan-writer notes section; the stencil tells the writer not to gather those.
