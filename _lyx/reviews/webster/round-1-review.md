---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/reedengine/bar.go:73
    summary: statusFormatSessions ends with the right-aligned time segment written twice, so line 2 of the bar renders the clock twice and no test counts it
    fix: In internal/reedengine/bar.go, drop the second "#[nolist align=right]%H:%M " from the end of the statusFormatSessions concatenation, leaving followButtonFormat + "#[nolist align=right]%H:%M " as at the merge base; in TestBar_LineTwoListsSessionsPrimeFirst (internal/reedengine/bar_integration_test.go) assert that the expanded line 2 holds the time segment once, or add that count to the format-string check in bar_test.go.
    origin: lens:code-correctness
---

### [BLOCKING] Bar line 2 draws the time twice

**Location:** internal/reedengine/bar.go:73
**Issue:** The rebuilt `statusFormatSessions` ends `followButtonFormat + "#[nolist align=right]%H:%M " + "#[nolist align=right]%H:%M "`; the merge base (`e14e9b495`, `bar.go:64`) had the time segment once, and card 8's intent says the time stays where it is.
Every session row therefore shows the clock twice.
Neither `TestBar_LineTwoListsSessionsPrimeFirst` nor `TestStatusFormats_SessionLoopsTakeOneFormat` counts the time segment, so the suite passes with the defect in.
Four lenses reached it independently (correctness, affected-set, tests, pattern-fit).

## Rejected

- No test for the `lyx reed attach` floor check (code-tests): card 7's intent states no test injects a version line there and `TestTmuxFloor` pins the check, as the decision record's acceptance criterion says.
- No test for `planglyph.Delta` returning the floor error (code-tests): card 5's intent states the same, with `TestCheckFloor` pinning the check.
- `TestAcquireWeftWriteLock_SignalsOnlyItsOwnPath` tests the test seam, in-package (code-tests): card 10 specifies exactly this untagged in-package test.
- `launchBlockedOnLock`'s closing non-blocking `select` is weak (code-tests): it cannot fail wrongly, the signal-before-wait design is the plan's, and a verb that skipped the lock would fail the earlier `done` case.
- `PATTERN-gitexec-checked-call.md` and `PATTERN-leaf-packages.md` do not mention gitexec's new `logger` and `toolfloor` imports (code-pattern-fit, informational): `gitexec` is not a leaf entry, nothing is violated, and `PATTERN.md` gains no line by decision.
- Package-level `versionProbe` and `tmuxVersionProbe` swapped by tests (code-pattern-fit): the tests that swap them are serial and say why, and the plan's decision names this seam.

## Focus-area results

- toolfloor: constants stay `2.43.0` and `3.4`, the package imports the standard library only, spawns and logs nothing, and the audit records `allow-set-title` (tmux 3.5) as an optional above-floor feature with reed's Warn-and-continue fallback; not a defect.
- gitexec and reed floor checks: gitexec probes once per process and reed once per resolved path, each caching only parsed answers, retrying a probe that could not run, and admitting an unparseable line with one Warn; not a defect.
- `CherryPick`: no `--empty=drop` remains in code, tests or docs beyond the audit's history line; it skips an already-carried change and refuses an empty-from-start commit; not a defect.
- `pattern/PATTERN-leaf-packages.md` lists `internal/toolfloor` and `PATTERN.md` has no diff; not a defect.
- creel deliver and merge-lock tests wait on a probe or lock-wait signal, not a sleep; no duplicated coverage found.
- Gate: `lyx gate test` over `toolfloor`, `gitexec`, `reedengine`, `gitrepo` and `fabricengine` passed; the tmux and integration tiers were not run here, and the duplicate exists in those tiers' only coverage of line 2.
