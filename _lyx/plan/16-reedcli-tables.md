# Card 16 — reedcli-tables

**Edit:**
- `reedcli/watchdog_test.go`
- `reedcli/spawnwatchdog_test.go`

**Uses:**
- `internal/reedcli#planReapCycle`
- `reedengine/spawnwatchdog_test.go`

**Intent:**
Collapse reedcli's per-scenario reap-cycle functions into one table and delete its duplicate watchdog-spawn tests (#325 §5, §6).

`watchdog_test.go`: the `TestPlanReapCycle_*` functions, each looping `planReapCycle` over a short cycle sequence, become one `TestPlanReapCycle` table whose rows each name a scenario and carry its full cycle sequence (live and missing sets per cycle, in-flight and hub-probe inputs where a scenario uses them) with the expected reaped names, remaining names and counter map after each cycle.
Every scenario keeps its exact sequence and its assertions; a scenario whose assertions the table shape cannot hold (the counter-map hygiene check, the hub-probe refusal) stays a row with a per-row check function rather than a separate test.
`equalStringSlices` goes; comparisons use `slices.Equal`.

`spawnwatchdog_test.go`: `TestEnsureWatchdogSpawned_SuppressedReturnsWithoutSpawning` and `TestEnsureWatchdogSpawned_EmptyHubPathReturnsWithoutSpawning` repeat `reedengine/spawnwatchdog_test.go`'s two early-return tests through a one-line delegation; keep one of them as the delegation's wiring proof and delete the other.
`TestWatchdogLockPath_IsHubScratchDirLockFile` compares a constant with itself: delete it.

The leaf count changes on purpose (functions become subtests, duplicates go); list the before and after names in the commit body.
Apply D10 to every touched test.

**ImpactSummary:** Two untagged reedcli test files restructure; reap-cycle scenarios and their sequences are preserved as table rows, two duplicate tests are deleted.
