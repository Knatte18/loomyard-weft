---
round: 1
ledger:
  - key: implementer-spec-stale-loom-implementer-sentence
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised one finding (F1): `contracts/specs/implementer-spec.md` kept the old `loom.yaml` / `loom.implementer` sentence beside the new `tapestry.implementer` one.
The fix is a deletion and an antecedent restore, with the result fixed by the finding text.
