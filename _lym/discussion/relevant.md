# Relevant files

- `docs/research/speed/testmap-use-cases-2026-11-18/report.md` — the study the three uses come from, with the maps' bounds and figures
- `internal/testcover/doc.go` — the map format, block keys, function keys, storage directories and the mutant machinery
- `internal/testcover/coveragemap.go` `Dir`, `ReadMap` — the map record and its per-package reader the new reader builds on
- `internal/gatecli/redundancy.go` `redundancyStep.run`, `redundancyStep.judge`, `redundancyStep.mappedPackages` — the advisory report step the reach and mutation steps copy
- `internal/gatecli/redundancyreport.go` `RedundancyReport`, `ReadRedundancyReport`, `RedundancyReportMatches` — the report file and the HEAD-tree match the new reports copy
- `internal/loomcli/redundancynote.go` `redundancyReportNote`, `renderRedundancyReportNote` — renders the rubric note the reach and mutation notes copy
- `internal/gatecli/failurereport.go` `failedRun` — builds the failure report the triage lines join
- `internal/gatefixer/prompt.go` — the fixer reads the report file its caller names, so triage lines reach it through the failure report
- `contracts/stencils/loom/loom-rubric-code-review.md` — the test-economy findings and the redundancy report marker the new markers sit beside
- `internal/gateslot/tiers.go` `ReportRedundancy` — the report names a gate key's reports may hold
- `internal/gateslot/template.yaml` — the template keys that list reports
- `pattern/PATTERN-test-speed.md` — the coverage map and redundancy report rules D3 changes
- `pattern/PATTERN-gate-slot.md` — slot and verify rules the steps run under
