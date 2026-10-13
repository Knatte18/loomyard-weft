# Lens review

- test-economy, LOW, card 2 tackler rows repeat the shared admit's cases: fixed, the tackler test keeps only the in-effect admit, the added-pause refusal and the other-row refusal.
- test-economy, LOW, card 7 `TestReport_RoundTrip` pins a dead field: fixed, listed on card 7 with its `Deviations` equality dropped.
- criteria-coverage, MEDIUM, the hub-pause criterion has no test: rejected. `internal/loomcli/drive.go:757-769` (`goDriver.holdPause`) holds before every re-step whatever the consume returned, and `internal/loomcli/drive_test.go:528-538` drives it with covering pause answers. Card 2 changes nothing after `Admit`, and the admitted consume returns the same nil an ordinary one does.
- criteria-coverage, MEDIUM, no row reaches deduplication or sorting of the deviation list: fixed, card 7's recover-batch case feeds three symbols across two files out of order and wants the deduplicated, sorted pair.
- criteria-coverage, MEDIUM, the landed-parent-commit rule is unpinned: fixed, card 5's alignment claims pin "subject and diff" and "already makes one of this run's decisions".
- criteria-coverage, LOW, tie and unknown order unpinned: fixed, card 5 pins "tie" and "unknown order".
- criteria-coverage, LOW, the Master summary claim is unscoped: fixed, card 8 adds a `wantNone` of the old summary phrase and of the three-key report wording.
- criteria-coverage, LOW, no row parses a report without `deviations`: fixed, card 7 names `TestReport_RoundTrip`'s "OK with empty deviations" row, which `omitempty` now writes with no key.
- row-reachability: no report; the fork died on an API error, and the lens was not re-spawned.
- forced-edits: no report; the fork died on an API error, and the lens was not re-spawned.
