---
round: 1
focus:
  - "decision-record.md D3 and D4: settle whether the check, appended by the plan index and not inside `planparser.ValidateFormat`, really stays out of `planglyph`'s dispatch validation, which D4 says calls `planparser.ValidateFormat`; verify against `internal/planglyph/index.go` and `internal/planparser`."
  - "decision-record.md D3 attack surface: settle whether each exemption or skip (`language: none`, the `testdata` segment, done cards at the format gate, no skip at rework) states what it lets through and what bounds it, and whether the `testdata` exemption can be used to slip a real test file past the check."
  - "decision-record.md D1 and D5: settle whether the premise that a `plan:` handle's unit can never be a `.go` file, and that a `<dir>_test` unit binds, holds against `handle-malformed` and `internal/planglyph/glyphchain_test.go`."
  - "decision-record.md D2 and D6: settle whether D6's claim that nothing in the fork's path depends on the plain test-file path holds once D2's `**Uses:**` rule is the only placement signal, and whether D6 and the second Open risk agree with each other."
  - "decision-record.md Scope and Acceptance criteria: settle whether every Scope item has a matching criterion where one is needed, and whether the Constraints (`PATTERN-refusal-way-forward`, `PATTERN-test-economy`) are consistent with the Scope's test-budget and doc obligations."
---

Seed focus drawn from reading both artifacts and the discussion spec.
Nothing above targets a missing "Notes for the plan writer" subsection, rejected alternatives in `decision-record.md`, or call-site enumeration.
