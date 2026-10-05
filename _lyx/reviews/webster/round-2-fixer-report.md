# Round 2 fixer report

All five findings fixed, one commit each, nothing pushed.

## Changed

- **F1** (BLOCKING, `eb45acf29`): re-broke the hard-wrapped comments the range adds, moves or edits into one sentence per line, with no wording changes.
  Files: `internal/webstercli/cli_tmux_test.go`, `internal/webstercli/cli_integration_test.go`, `internal/reedcli/watchdog_integration_test.go`, `internal/reedcli/watchdogreap_integration_test.go`, `internal/reedcli/smoke_resume_test.go`, `internal/reedcli/smoke_test.go`, `internal/reedcli/smoke_dotfill_test.go`, `internal/reedengine/contract_integration_test.go`, `internal/reedengine/ensuresession_integration_test.go`, `internal/reedengine/mouse_boot_integration_test.go`, `internal/shuttlecli/modelpin_test.go`, `internal/shuttleengine/claudeengine/testmain_test.go`, `internal/burlerengine/smoke_cluster_test.go`.
  One comment the review named was left as it is: the break in `internal/lyxcwd/enforcement_test.go` sits before a `so` clause with its own subject and verb, which the line-break rule permits.
- **F2** (LOW, `66e5a5a69`): joined the sentences still split at non-clause boundaries in `internal/reedcli/watchdog.go`, `cmd/testtiming/main.go`, `cmd/testtiming/redundancy.go`, `internal/gitkit/reexecguard.go` and `internal/reedengine/doc.go`.
- **F3** (LOW, `5401fb0c5`): `EndSessionByName`'s doc in `internal/reedengine/overlay.go` now states the removal is bounded best effort and a file still answered after the wait stays, with no error.
- **F4** (LOW): `internal/reedengine/socketfile_test.go` gains `TestRemoveSocketFileOnceGone` (stale socket removed; live listener kept after a short wait, no error), and the listener helper became a package-level function shared with `TestRemoveStaleSocket`.

- **F5** (MEDIUM, added after the verify gate failed): `TestSmokeSelvageSurvivesCtrlCAtIdlePrompt` in `internal/reedcli/smoke_selvage_keepalive_test.go` now waits for Selvage's prompt to render before sending Ctrl-C.
  Before the fix it failed about one run in five alone; after it, ten runs in a row passed.

## Verification

`go build ./...` and `go vet` under every tag set are clean.
`go test` passes for `./cmd/...`, `internal/reedengine`, `internal/reedcli`, `internal/webstercli`, `internal/shuttlecli`, `internal/gitkit/...` and the testkits, and `-tags integration,tmux` passes for `internal/pairteardown` and `internal/reedengine`.
`gofmt -l` lists only files this range does not touch.

## Deferred

Nothing.
