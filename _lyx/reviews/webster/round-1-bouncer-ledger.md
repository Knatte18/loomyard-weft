---
round: 1
ledger:
  - key: isboxrule-labelled-shortcut-closes-box-from-below
    rounds: [1]
    status: resolved
  - key: soft-cycle-smoke-handoff-shell-assertion-too-loose
    rounds: [1]
    status: resolved
  - key: spec-field-docs-fixed-column-wrapped
    rounds: [1]
    status: resolved
  - key: resume-go-missing-file-header
    rounds: [1]
    status: resolved
  - key: transcript-stat-error-worded-as-missing
    rounds: [1]
    status: resolved
  - key: prepare-doc-lists-two-validations
    rounds: [1]
    status: resolved
  - key: defer-turn-end-requalify-untested
    rounds: [1]
    status: resolved
---

Round 1 was the first round, so there was no prior ledger.
The review approved with seven findings: four LOW and three NIT.
All seven were fixed in their own commits, `a0417f160` through `f5ef95742`, and each was re-checked against the diff.

- The labelled-rule shortcut is now limited to the top rule (`isTopBoxRule`), and near-miss tests pin it.
- The soft-cycle smoke test requires the test's own shell, `SOFTBG` or `sleep 900`.
- The new and changed `Spec` comments use semantic line breaks.
- `resume.go` and its test have file headers.
- A transcript stat error is worded apart from a missing transcript.
- `Prepare`'s doc names all four validations.
- A test shows that a DEFER turn end alone never re-arms the soft trigger.

No finding remains open.
