# Gate tiers run go test with no memory cap

`gate-tier-memory-cap` · Note · bug, gate

## Seen (2026-11-17)

The diagnosis of `redundant-mutant-memory-cap` found that the gate tiers run `go test` with no memory cap either (`internal/gatecli/tierrun.go`), and the gate watchdog samples CPU only (`internal/gatecli/watchdog.go`).
A test that allocates without bound in a normal gate run can freeze the machine the same way a mutant did.

## Idea

Once `redundant-mutant-memory-cap` has built the per-run memory cap in `internal/proc` and `internal/testcover`, put the gate tiers' `go test` runs under the same cap, with a per-tier setting, and have the watchdog report memory beside CPU.
