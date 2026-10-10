# Card 2 — gatecli-tied-child

**Delete:**
- `internal/gatecli/child_linux.go`
- `internal/gatecli/child_unix.go`
- `internal/gatecli/child_windows.go`

**Create:**
- `internal/gatecli#runTiedChild`
- `internal/gatecli#terminatingSignalSet`

**Edit:**
- `internal/gatecli#runTest`
- `internal/gatecli#catchTerminatingSignals`
- `internal/gatecli/doc.go`

**Uses:**
- `internal/proc#StartTied`

**Intent:** `lyx gate test` uses the tie from `internal/proc` instead of its own copy, as D2 states.
The three platform files go, and with them the platform-split `runChild` and `terminatingSignals`; one `child.go` holds their replacements under names the gate resolves uniquely.
`runTiedChild` calls `proc.StartTied` on `cmd` after wrapping the cancel so the group kill is logged as today ("gate test: killing go test process group" with the pid, and the fallback Warn), then waits through `Tied.Wait` and returns its error; `runTest` calls it where it called `runChild`.
`terminatingSignalSet` is declared once as SIGINT, SIGTERM and SIGHUP, and `catchTerminatingSignals` registers it; Windows delivers only the interrupt and ignores the rest, so one shared declaration replaces the two platform ones.
`doc.go`'s description of the child tie points at `internal/proc` instead of restating the Linux and Windows mechanics.
The existing `integration` scenario test covers the verb; this card changes no behavior.

**ImpactSummary:** Two call sites in `run.go` take the new names; the deleted files' symbols had no caller outside this package.

**Verify:** GOOS=windows go vet ./internal/proc ./internal/gatecli
