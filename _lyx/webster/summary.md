# Tender plan run: batch 01 done, batch 07 stuck

## Batch 01 (pauserecord-network-held): done

`record-batch 01` returned `status: done` with `head_sha` `3c5699680211f4f2b9f924e3bf7d9230265ecdf6`.
The scope guard listed four files changed outside the batch's own targets, all informational:
`internal/githubclient/transport.go`, `internal/orchengine/notice.go`, `internal/orchengine/watcher.go` and `internal/shuttleengine/conditionedsend_test.go`.

## Batch 07 (proc-process-listing): stuck

The batch holds cards 7 to 12.
Its fork's reply said every card was committed and a report with `status: OK` was written, ending at `5348618b43261dbfdac31795c5e8fda821e0adc3`.
The fork also reported that `TestTierPurity_UntaggedTestsSpawnNothing` fails on `time.Sleep` calls with unresolvable durations in two untagged test files that batch 01 committed, `internal/githubclient/githubclient_test.go:677` and `internal/selfreportengine/inbox_test.go:100`, and that it left them alone.
These are the fork's words, not something I checked.

`record-batch 07` refused with `batch_failed`: the done-check found plan handles that no longer resolve.
Those handles name `internal/pausecli` and `internal/pauseengine` symbols for cards 8, 9 and 10, such as `pausecli#resumeCmd`, `pauseengine#SessionStatus` and `pauseengine#Revive`.
`recover-batch 07` was run once and refused with the same `batch_failed` finding, so no further recovery is attempted.
I did not check whether card 12's rename of those packages is the cause.

The run stops here with one of two batches done.
The plan-level verify gate has not run.
