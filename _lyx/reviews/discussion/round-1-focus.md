---
round: 1
focus:
  - "D7 (`webster reset` remote update): check that the Decision's bound and Constraints settle what the force update can discard, including a remote tip that moves between the pre-mutation fetch and the lease-guarded update, and a reset whose local HEAD has already diverged from the remote before the reset."
  - "D3 and D4 (guards downgraded or added): check that each states what it lets through or refuses and what bounds it, and that D3's dispatch-only relaxation cannot let a Delete target the run never meant to lose pass unnoticed, for example a missing path caused by a card editing the wrong file."
  - "D1, D2 and D4 consistency: D4 reuses D2's check at `record-batch` and `recover-batch` time, while D2 is scoped to not-yet-completed cards at dispatch; check that the scope D4 needs is stated and does not contradict D2's."
  - "D1 against Scope and Acceptance criteria: Scope says no cycle check and no change to Webster's sequencing; check that the acceptance criteria and the stated reasoning for the cycle case and the card-order guarantee follow from the Decision, and that including **Delete:** targets in D1 matches the stated rule."
  - "D5 and D6 premises: check that the Decisions' claims about when a batch has no record, and about `failed`, `dead`, `done` and non-terminal batch states and what `recover-batch` renders from, are consistent with each other and with the Acceptance criteria."
  - "D8 and D9: check that each way forward and outcome statement the Decision promises is one the Decision itself makes available without a new verb, flag or config key, and that D9's dependence on the open question Q11 is carried into the decision record rather than left only in the support log."
  - "Relocation: look for deliberation narrative or history in `decision-record.md` that carries no requirement for the plan writer, but first extract any constraint it holds into Decisions or Constraints."
---

Focus is on unbounded guards and edges (D3, D4, D7), cross-decision consistency (D1, D2, D4, D5, D6) and the premises the Decisions rest on.
None of the entries asks for exact signatures, call-site lists or rejected alternatives in `decision-record.md`.
