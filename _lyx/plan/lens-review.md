# Lens review

All four forks returned a report.

- criteria-coverage + row-reachability, BLOCKING, card 11: `classifyHang` checked `SIGQUIT: quit` before the watchdog line, so a real quit never named its window. Fixed: the watchdog line is checked first, and the watchdog rows' logs carry both lines.
- criteria-coverage, MEDIUM, card 3: embedded promoted fields and interface fields were unreached. Fixed: the fixture gains an embedded struct and an `any` field, each with a row.
- criteria-coverage, LOW, card 3: the replaced-map-value bound was untested. Fixed: the map row replaces the value with different known fields under the same key.
- criteria-coverage, LOW, card 10: the nested-gate row names the test process rather than a real `lyx gate test`. Rejected: D7 defines the skip as the variable naming a live ancestor, and that is what the row pins; the spin row covers a real gate run.
- test-economy, MEDIUM, card 3: `TestRun_UnknownKeysSurvivePersist` repeated `TestRun_UnknownTopLevelKeyBeforeRead` and `TestRun_ExternalWriteSurvivesPersist`. Fixed: card 2 flips the former in place, card 3 extends both with kept-key assertions and deletes `TestRun_StrayExternalKeyDestroyed`, so no new shedengine test remains.
- test-economy, LOW, card 3: the absent-file row and the strict-read part of the status row repeated existing coverage, and the status row imported `shedengine`. Fixed: both dropped, and the round-trip row uses the local fixture.
- test-economy, LOW, card 1: the new row overlapped existing rows. Fixed: `saveExact` is set on the tier-and-type row and the set-status kept-keys row, with no new row.
- test-economy, LOW, card 11: the exit code 255 row repeated the 137 row. Fixed: dropped.
- test-economy, LOW, card 10: the gatecli budget needed 4, not 5. Fixed.
- forced-edits, MEDIUM, card 5: `internal/shedrun/doc.go` describes the retired `parent` key and ParseSeed's strictness. Fixed: added to card 5 with both rewordings.
- forced-edits, LOW, card 10: `pattern/PATTERN-daemon-wakeups.md` lists each wait. Fixed: added.
- forced-edits, LOW, card 4: the `content` field comment names the old test by name. Fixed: retargeted by hand.
- forced-edits, LOW, card 12: `NewVerifyGate`'s doc describes the findings. Fixed: added.
- forced-edits + row-reachability, LOW, card 13: the log-path check would read an empty `Log`. Fixed: the row hands the built copy to the fake and asserts on it.
- forced-edits, LOW, card 10: `docs/overview.md`'s gate row. Fixed: added.
- row-reachability, MEDIUM, card 10: the logcapture row ran beside parallel tests and needed Info capture. Fixed: `TestWatchdog` is not parallel and uses `CaptureVerbose`.
- row-reachability, LOW, card 12: the shared `last line` assertion. Fixed: the watchdog row's log ends with it.
- row-reachability, LOW, card 11: the marker logs need a failing identity. Fixed: each ends with `failingPackageLog`.
- row-reachability, LOW, card 9: the `/sleep` path suffix is host-dependent. Fixed: it asserts the child's pid with a non-empty path.
