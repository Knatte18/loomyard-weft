---
round: 2
focus:
  - "In internal/shedadapters/bouncer.go, the `escalate` path that returns Stuck when `readEscalation` fails: does its Reason name a concrete way forward, does contracts/specs/refusal-spec.md carry a row for that refusal, and does an untagged test in the shedadapters package seed a malformed escalation record and assert the halt and its Reason (PATTERN-refusal-way-forward)?"
  - "Any other Stuck or refusal path that the committed diff adds in internal/shedadapters or internal/loomrecipe: does each name its way forward, have a refusal-spec row, and have a reaching untagged test in the same commit?"
---

Focus restates the sites to check; it caps no severity.
