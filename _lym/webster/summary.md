# Background pusher for run records and code branches

All four batches in the card list reached `done`.
Batch 01 was already done before this session; the digests below are from this session.
No digest listed a departure or a rename.

## Batch 04: pairteardown-prime-handover

Cards 4, 5 and 6 were committed by an earlier session that left no report.
The fork added one fix commit for a pending `internal/loomcli/pusher.go` edit.
Head `17341f7c2`.
Scope guard deviations: `cmd/lym/testonlycode_linux_test.go`, `internal/loomcli/pusher.go`, `internal/loomcli/pusher_test.go`,
`internal/runpusher/pushuntillanded.go`, `internal/runpusher/run.go`, `internal/runpusher/run_test.go`,
`internal/runpusher/start.go`, `internal/runpusher/start_test.go`.

## Batch 07: transitions-start-pusher

Transitions and records commits now commit and start the pusher, and never push.
Head `1a9cdd29e`.
Scope guard deviations: `cmd/lym/testonlycode_linux_test.go`, `internal/adoptcli/adopt_integration_test.go`.

## Batch 08: restarts-start-pusher

The dobby's tick and a resume start the pusher.
Worktree-Teardown pushes the pair before removing it.
The PATTERN entries and docs state the Decisions.
Head `b74a788e5`.
Scope guard deviations: `cmd/lym/testonlycode_linux_test.go`, `internal/battencli/commitstatus.go`,
`internal/battencli/lifecycle_integration_test.go`, `internal/battenshed/deps.go`, `internal/battenshed/teardown.go`,
`internal/battenshed/teardown_test.go`, `internal/loomcli/resume_test.go`.
