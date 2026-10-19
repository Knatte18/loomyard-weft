# ubuntu-2404-support: git 2.43 and tmux 3.4 floor

All five batches reached `done`.

- **Batch 01 (`01-lyxtemp-short-run-names`, head `fec538da8`):** The `record-batch` done-check failed because `internal/toolfloor#TestCheck` did not resolve, and one `recover-batch` run completed it with `status: done`. Deviations: `internal/toolfloor/testmain_test.go`, `internal/toolfloor/toolfloor.go`. No departures.
- **Batch 03 (`03-cherrypick-without-empty`, head `cfe202206`):** `CherryPick` runs a plain pick, skips an already-carried change and refuses an empty commit. No deviations or departures.
- **Batch 04 (`04-gitexec-floor-check`, head `7b2d037a8`):** gitexec checks git's floor once per process, and `planglyph.Delta` checks it before quarry's git spawn. No deviations or departures.
- **Batch 06 (`06-reed-tmux-floor-check`, head `9c4a8eb70`):** reed checks tmux's floor before every tmux spawn, the attach command checks it before its handover, and the session row is two single-format loops with the prime's session first. Deviations: `internal/reedengine/bar_integration_test.go`, `internal/reedengine/bar_test.go`, `internal/reedengine/tmuxfloor_test.go`. No departures.
- **Batch 09 (`09-creel-deliver-marked-probe`, head `fdbc66c22`):** Creel's deliver tests wait for a probe that saw the post, and the merge-lock tests wait for the verb's own lock-wait signal. No deviations or departures.

The scope guard's only findings were the deviations listed above, which are informational. The plan-level verify gate runs after this turn.
