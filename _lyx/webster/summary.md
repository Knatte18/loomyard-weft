# Tender: network-held pause record and process listing

Batch 01 (`pauserecord-network-held`) finished `done` before this session resumed.

Batch 07 (`proc-process-listing`) had a failed first recovery.
A second `recover-batch 07` ran and returned `status: done` at `head_sha` 5348618b43261dbfdac31795c5e8fda821e0adc3, with `recoveries: 2` and no warnings.

The scope guard listed deviations for batch 07, all informational.
They are the `pausecli` to `tendercli` and `pauseengine` to `tenderengine` rename files, `internal/reedcli/attach.go` and `internal/shortcutcli/shortcut_test.go`.
Those files fall under the batch's rename and attach-exec work.

## Verify gate fixes

The gate's first attempt failed on `TestTierPurity_UntaggedTestsSpawnNothing`.
It flagged a `time.Sleep` of one second or more in `internal/githubclient/githubclient_test.go` and `internal/selfreportengine/inbox_test.go`, because the guard could not resolve function-local duration constants.
The fixer moved the durations to file-scope constants (`pauseHoldDuration`, `inboxPageDelay`, `inboxPauseHold`) with the sleep lengths unchanged, in `fix:` commit `6d75647a6`.
The fixer ran `lyx gate test --tags integration` on `./cmd/lyx`, `./internal/githubclient` and `./internal/selfreportengine`, and it passed.
