---
round: 2
focus:
  - "internal/gatecli/dotnetscope.go, namesProject: settle whether a tier command that names only a built assembly (a .dll or .exe path) is treated as reaching every selected project, and whether matching compares project paths rather than bare file names."
  - "internal/gatecli/run.go, runTest: settle whether a test drives the dotnet scoper path that prints the unreached-project lines and still exits 0."
  - "internal/gatecli/tiers.go, planTierRuns: settle whether every refusal it raises ends with the role clause doc.go promises, and whether the run-whole notice can print before a later refusal."
  - "internal/gatecli/dotnetscope_test.go, TestUnrunProjectLines: settle whether the zero-invocation, duplicate-selection and backslash-path branches of the helpers have coverage."
---

The focus lists where the round 1 findings touched the code, so the fixes get a fresh read.
