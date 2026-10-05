---
round: 3
focus:
  - "Go comments the range adds or moves, in `internal/webstercli/cli_tmux_test.go`, `internal/webstercli/cli_integration_test.go`, `internal/reedcli/watchdog_integration_test.go`, `internal/reedcli/watchdogreap_integration_test.go`, `internal/reedcli/smoke_resume_test.go`, `internal/reedcli/smoke_test.go`, `internal/reedcli/smoke_dotfill_test.go`, `internal/reedengine/contract_integration_test.go`, `internal/reedengine/ensuresession_integration_test.go`, `internal/reedengine/mouse_boot_integration_test.go`, `internal/shuttlecli/modelpin_test.go`, `internal/shuttleengine/claudeengine/testmain_test.go`, `internal/lyxcwd/enforcement_test.go` and `internal/burlerengine/smoke_cluster_test.go`: settle whether every added comment line follows PATTERN-comment-line-breaks, one sentence per line with breaks only at clause boundaries and no column wrapping."
  - "Sentence breaks inside Go comments in `internal/reedcli/watchdog.go`, `cmd/testtiming/main.go`, `internal/gitkit/reexecguard.go` and `internal/reedengine/doc.go`: settle whether any sentence is split across lines at a point that is neither an independent-clause boundary nor a semicolon."
  - "`EndSessionByName` in `internal/reedengine/overlay.go` and `removeSocketFileOnceGone` in `internal/reedengine`: settle whether the doc comment matches the bounded, best-effort socket-file removal, and whether the branch where the wait elapses with a live listener is covered by a test."
  - "`TestSmokeSelvageSurvivesCtrlCAtIdlePrompt` in `internal/reedcli/smoke_selvage_keepalive_test.go`: settle whether the test waits for the Selvage pane's shell prompt before sending Ctrl-C, and whether it passes repeatedly (`-count`) under the `tmux` tag."
---

Round 3 re-checks the same four areas against the current commits.
