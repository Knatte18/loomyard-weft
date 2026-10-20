---
round: 1
focus:
  - "internal/reedcli/watchdogreap_integration_test.go, `goneWithin` near line 282 and the `waitProcessGone` calls near lines 170 and 234: do the success waits that still derive from `ReapTimeout`, `reapForceKillGrace` or a fixed 20 s stay sound under the stall the board entry describes, or is a slow-host wait left that the entry's expectation covers?"
  - "The fresh-session wait in `LoopStaysLiveDuringReap` (`freshEng.SessionName()`): does it wait on something the watchdog loop does, so a stalled cycle could miss the old `DiscoveryCycle*10` window, and does the replacement bound keep what the step proves?"
  - "`cancelDeadline` and `dispatchStart` near lines 348-353: does raising the preceding waits to 30 s change what the cancellation-independence assertion measures or its failure message?"
  - "`_lyx/landing/summary.md` against the diff: the claim that every success wait in every step now uses `reapSuccessWait`, and that the bounds left short only assert time running out."
  - "The comment on `reapSuccessWait`: does it match the repository's comment conventions and state what the constant bounds correctly?"
---

The diff is one test file: a new constant and seven replaced wait bounds.
The verify gate decides build and test results, so focus is on scope against the board entry and on waits the change left derived from the cadence.
