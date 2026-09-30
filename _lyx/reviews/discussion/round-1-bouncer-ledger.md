---
round: 1
ledger:
  - key: plan-writer-note-overstates-confinement
    rounds: [1]
    status: resolved
  - key: goal-overstates-refusal-untouched
    rounds: [1]
    status: resolved
  - key: byte-identical-constraint-misdescribed
    rounds: [1]
    status: resolved
  - key: d2-commit-archaeology
    rounds: [1]
    status: resolved
  - key: generic-line-break-constraint
    rounds: [1]
    status: resolved
---

Round 1 was the first round; there was no prior ledger.
The reviewer raised five findings against `decision-record.md`, and the fixer resolved all five in the same round.
The MEDIUM one (the plan-writer note claimed the change was confined to `remove.go`, `doc.go` and the `remove_*` tests while two other tests pin the old order) closed with a D6 bullet deciding both tests' inverted expectations.
The two LOW ones (the Goal's unscoped "refused `Remove`", and a constraint that read as contradicting D3) closed by scoping the Goal and restating the constraint as base texts only.
The two NITs were relocations: the D2 commit archaeology went to the support log, and a repo-wide line-break rule left Constraints.
No finding remains open.
