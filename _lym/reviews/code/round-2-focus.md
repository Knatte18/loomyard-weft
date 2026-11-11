---
round: 2
focus:
  - "internal/gatecli/testmappackage.go `stampEnvironment` and `mappedTierConfigs`: does a test pin that a changed tier command or env value, or a changed hub env.yaml variable, changes the stamp digests, and that a changed tier parallel value does not?"
  - "internal/testcover/declcompare.go `compareTestSources`: does a new test file importing `testing`, or a removed one, avoid a miss while an import outside the standard library and the linked set still misses, and does the selector then select the new Test functions?"
  - "internal/testcover/testselect_test.go `TestSelectorSelect`: does each of the seven stamp fields have a miss row, and is the miss detail asserted to name the differing field?"
  - "internal/testcover/testselect.go `mapMiss`: does a build-failure record beside test entries leave the map usable, as the decision record's selector rules say, and is the choice pinned by a row?"
  - "internal/testcover/testselect.go `readTargetFile` and `environmentMiss`, internal/gatecli/coveragescope.go `openSelector`, internal/testcover/declcompare.go package-clause branch: do the unreadable-target, unreadable-toolchain and package-clause-change paths each have a test row?"
---
