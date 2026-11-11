---
round: 1
focus:
  - "internal/fabricengine/hubworkspace.go: does the hub file list every pair at any depth under beams, and still the prime and Bolt, with no duplicates or omissions?"
  - "internal/fabricengine/hubworkspace.go and internal/boardengine/handles.go: is a folder named `<handle>: <slug>` only when the pair's board entry belongs to a project, and the bare slug otherwise, including when the board lookup fails or the entry is missing?"
  - "internal/fabricengine/hubworkspace.go: are beam workspace files left unchanged, and does the handle come from the board at regeneration rather than a cached value?"
  - "cmd/lym/main.go and cmd/lym/prerunseams_test.go: does the new seam wiring keep the existing seam installation behavior, and do nothing extra when boardengine is unavailable?"
  - "internal/fabricengine/hubworkspace_integration_test.go and internal/boardengine/handles_test.go: do the tests cover nested-beam pairs, project-less pairs and handle naming, without duplicating one another?"
  - "docs/overview.md, internal/fabricengine/doc.go and internal/boardengine/doc.go: do the doc changes match the behavior the diff lands, and does the commit summary at _lym/landing/summary.md, if present, bear out against the diff?"
---

Seed focus drawn from the board entry under `## Goal`, the only settled scope; the record's other sections are empty.
