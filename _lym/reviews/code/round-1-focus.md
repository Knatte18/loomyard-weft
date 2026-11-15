---
round: 1
focus:
  - "internal/gatecli/testmappackage.go and testmap.go: does the build follow design item 6 — one process per test with block coverage, each profile read once, queued tests run side by side, one write per package, test binaries built ahead?"
  - "internal/gatecli/tiers.go and internal/gateslot: does the gate always run whole packages, with no map-based selection left on any path, config key or template default (design item 4)?"
  - "Removal of the dobby upkeep pass and the `test_map_upkeep` key (design items 1 and 3): is any trigger, config reader, doc or caller left that starts a build or trim automatically, and is anything outside the stated removal scope deleted?"
  - "internal/gatecli/testmapwait.go and testmap_integration_test.go: does the on-command build handle failure paths (interrupted build, stale or partial maps, concurrent runs) without leaving a corrupt map set?"
  - "Design item 7: does the build take the recommendations of the `testmap-miss-causes` report that lower the number of packages re-measured, and does the diff bear out the claims in `_lym/landing/summary.md` on this and on the timing of one full build?"
  - "pattern/PATTERN-test-speed.md and pattern/PATTERN-daemon-wakeups.md: do the edits match the declared PATTERN changes, and do docs the repository's constraints name for this kind of change land in the diff?"
  - "Deleted tests across internal/gatecli, internal/gateslot, internal/testcover: does each deletion follow removed behavior, with the behavior that remains still covered? Check the contained candidates in the redundancy report against their covering tests' assertions."
---

Seed focus from the board entry's design list, the diff stat and the commit subjects.
