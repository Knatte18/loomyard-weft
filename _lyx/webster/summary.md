# Hibernate trigger: tender lift, hibernate engine and CLI

Both batches recorded `done` with no recovery.

## Batch 01-tender-lift-entry (head 61e42da0d)

Built cards 1–4.
`tenderengine` lifts one pause entry by identity and reports the entry a pause adds.
`tendercli` exports its hub wiring as `HubDeps`.
`internal/hibernateengine` gained the request file and the submit, cancel and status sequences, then the daemon's `Watch`: trigger wait, pause, confirmation, hibernate and wake.
The scope guard listed these files as outside the batch's declared targets: the `internal/hibernateengine` request, verbs, watch and test files, and `internal/tendercli/cli.go`.
The fork also edited `cmd/lyx/daemonwakeups_test.go`, `pattern/PATTERN-daemon-wakeups.md` and `cmd/lyx/testdata/test-budget.yaml`.

## Batch 05-hibernate-cli (head 7a9ad9f0f)

Built cards 5–8.
The `lyx hibernate` module (`internal/hibernatecli`) wires the engine to the hub and spawns the daemon.
Spawned Claude sessions are denied the machine sleep commands.
`lyx p` and `lyx r` shortcuts cover pause and resume.
The orch role file routes an operator's hibernate request through `lyx hibernate`.
The scope guard listed `internal/hibernatecli/cli.go`, `cli_integration_test.go` and `testmain_test.go` as outside the declared targets.
The fork reported one card deviation: the `request` command's `Use` string omits the flags, because `TestCLIForm_EveryCommand` rejects the long form.

The plan-level verify gate is not run by Master.
