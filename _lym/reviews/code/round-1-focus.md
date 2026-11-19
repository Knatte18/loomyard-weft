---
round: 1
focus:
  - "internal/gatecli/mutatecheck.go, spreadEvenly: does the spread pick one mutant per changed function first, stay within mutation_max_mutants, and stay deterministic when functions outnumber or underrun the cap?"
  - "internal/testcover/mutant.go, logsOnly, blockLogsOnly, comparesOtherPlatform: do they skip exactly log-only statements and other-platform runtime.GOOS tests, with no legitimate mutation site dropped?"
  - "internal/gatecli/mutation.go, mutationreport.go, SummaryLine: does mutation.json carry every survivor, unjudged mutant, uncovered changed block and package not judged, and does the one summary line print every count the board entry lists?"
  - "internal/gatecli/mutate.go and run.go, mutationCall.run and runMutate: does a failure path (no go list, no coverage map, unbuildable package) degrade or refuse as the entry asks, and does the new step run only on keys whose reports list mutation?"
  - "internal/gateslot/config.go, template.yaml, tiers.go: is mutation accepted as a report step, listed last on every scope-diff key, and is the TestLoadConfig change covered by its Loosened-Check trailer with a measured cause?"
  - "contracts/stencils/loom/loom-rubric-code-review.md, internal/loomcli/mutationnote.go, internal/shedadapters/rubric.go: is the mutation_report marker filled with the counts and survivors, and read only when the report matches HEAD's tree?"
  - "Test economy: the diff adds no coverage for fillMutateAnswer, freshMaps, mutationCall.run and runMutate per the reach report; check whether the integration test covers them and whether any added test duplicates another."
  - "_lym/landing/summary.md against the diff, and the commit messages of the range: does any claim (the trial's 'bulk of survivors', counts) lack before and after figures or contradict the code?"
  - "Docs and PATTERN: does the diff land docs/overview.md, the package doc.go files and the pattern/ edits the change needs, and does each follow the repository's rules?"
---

Seed focus derived from the board entry's three changes (spread, skip rules, mutation.json plus summary line) and the
`mutation_report` marker, plus the reach report's uncovered functions.
