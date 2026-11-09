---
round: 1
focus:
  - "internal/gatecli/testmap.go and testmappackage.go: settle whether staleness (no map, differing Go file set or content at HEAD, age past test_map_max_age_days) and the --max cap match D5, including that dirty or locked skips are decided before any build and do not count against --max."
  - "internal/gatecli/testmappackage.go: settle whether the mid-package hash check discards the map, reports the package skipped and counts it against --max, while an unchanged-hash HEAD move keeps the map recorded with the HEAD read before the package."
  - "internal/gatecli/testmap.go: settle whether a timed-out test or build is ended with its whole process group, the wait after the kill is bounded by a fixed grace, and a timed-out test or undeclared tier is left out of the map."
  - "internal/gatecli and internal/testcover/exitrecord.go: settle whether the exit record is written on every exit path, zero count included, holding the process ID and the unbuilt-stale count through internal/state at the lymdirs path, and whether the failed-build and no-test maps follow the retry rules (failure map stale again after a HEAD move)."
  - "internal/gatecli, internal/fabricengine/codereads.go: settle whether the prime-worktree refusal names its way forward, whether git beyond the HEAD SHA read runs only inside internal/fabricengine, and whether the gate slot and per-package lock are held and released on every path."
  - "internal/gateslot/config.go and template.yaml: settle whether the new keys test_map_max_age_days, redundancy_build_timeout_min and redundancy_test_timeout_sec have the D5 defaults and load through configengine, and that no existing key changes meaning."
  - "Tests in testmap_integration_test.go, codereads_integration_test.go and internal/testcover: settle whether each acceptance criterion's behavior has coverage, that none launches a test binary or go test, and that no test duplicates another."
---

Focus follows the D5 decision and its acceptance criteria; the testcover and hubgeom changes are reviewed as supporting code.
