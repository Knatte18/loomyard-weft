---
round: 1
ledger:
  - key: envkit-fullenv-untargeted
    rounds: [1]
    status: open
  - key: overview-loom-keys-doc-not-edited
    rounds: [1]
    status: open
---

Round 1 raised two scope findings against the plan.
`envkit-fullenv-untargeted` (F1, MEDIUM) covers the shared `envkit.FullEnv` builder missing from the cards that add required `Env` fields.
`overview-loom-keys-doc-not-edited` (F2, LOW) covers `docs/overview.md`'s loom config key line going stale.
Neither is gating; both stay open until a fresh reviewer sees the fixed plan.
