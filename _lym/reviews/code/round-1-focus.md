---
round: 1
focus:
  - "internal/testcover/mutant.go and mutantverdict.go: does the verdict drop a candidate only when every mutant it kills is also killed by a covering test, and do unbuildable, timed-out or surviving mutants resolve without wrongly declaring redundancy?"
  - "internal/gatecli/redundant.go and redundantmutants.go: is the mutation check applied per candidate before listing, bounded by the candidate's blocks, with failure paths (build error, cancelled run, overlay cleanup) handled and reported?"
  - "internal/testcover/runner.go and sourcescan.go: do the overlay build, the mutant binary run and the scan-kit reader marking stay correct, leave no stray files, and avoid races with parallel runs?"
  - "Scope against the Goal: does the diff make the fixed verb drop the survey's sampled candidates, and does anything in docs, pattern/PATTERN-test-speed.md or doc.go change beyond what the Goal licenses or contradict the code?"
  - "Test economy: check TestPlanMutants (reported contained, covered by TestMutantPlan_WriteOverlay) by comparing assertions, and check new tests in redundant_integration_test.go and runner_test.go against existing coverage."
---

Seed focus only; the reviewer still reads the whole range.
