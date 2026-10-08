---
round: 2
focus:
  - "In `_lyx/plan/05-caller-scan-types.md`, settle whether every untagged test in any package that can reach `planglyph.Validate` or `planGatePass` with a plan that deletes or re-signs a member avoids spawning `go list`; check `internal/webstercli/cli_test.go` (untagged, `TestValidateCmd_Envelopes` writes a delete-order plan over a fixture repo) against the card's stated seam and its targets."
  - "In `_lyx/plan/05-caller-scan-types.md`, settle whether the card states how an identifier named like a subject that has no object in a type-checked file reaches card 4's scan through the re-signed `scanReferences(root string, files []string, subjects []coverageSubject)` without re-reporting the file's other name matches or double-counting references `typedReferences` already counted."
  - "In `_lyx/plan/04-caller-scan-import-path.md`, settle whether the Intent states what the exclusion of a method match whose selector's left side is an imported name lets through, and what bounds it, given the typed path of the following card."
---

Focus targets the plan's testing seam and the two scan-narrowing mechanisms; fidelity, order and verify coverage need no special attention.
