---
round: 1
focus:
  - "internal/gatecli/redundant.go and internal/testcover/verdict.go: does the candidate list follow the judgment of verdict.go, with `//testtiming:keep` and spawn-marked tests neither candidates nor covers, and are failure paths (missing map, unknown package, unreadable map) handled?"
  - "internal/gatecli/redundant_integration_test.go and internal/testcover/verdict_test.go: test economy. Does any test duplicate another's coverage, and does the integration test cover the verb's behavior beyond the pure JudgeMap test?"
  - "docs/research/speed/coverage-map-pilot-2026-11-12/report.md: does each claim (build outcome, wall/CPU/slot figures, full-set estimate as a range with its assumption, at most five candidates read, at most two commits compared, verdict section) bear out against the data files and the cap in the Goal?"
  - "Scope: does the diff stay inside the Goal's cap and its Not-in-scope list (no other package's map, no test cut or folded, no change to map format or upkeep rules), and are the edits to docs/overview.md, internal/gatecli/doc.go, internal/testcover/doc.go and pattern/PATTERN-test-speed.md licensed by adding the verb?"
  - "Repository rules: do the new verb and its docs follow the constraints document and the PATTERN.md and pattern/ conventions for the kind of change, and do the doc comments follow the repository's comment conventions?"
---

Round 1 seed: the Goal is a pilot report plus, if no verb existed, the smallest verb for the candidate list.
The diff adds `lym gate redundant`, so the verb's correctness and test economy come first, then the report's fidelity to the Goal's cap.
