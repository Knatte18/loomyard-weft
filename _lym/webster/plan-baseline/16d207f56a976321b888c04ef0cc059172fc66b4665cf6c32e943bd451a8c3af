# Card 12 — gate-run-in-process

**Edit:**
- `internal/gatecli#testRequest`
- `internal/gatecli#execution.runWithSlots`
- `internal/gatecli#execution.report`
- `internal/gatecli#execution.reportBlocked`
- `internal/gatecli#TestGateTest_Scopes`

**Create:**
- `plan:internal/gatecli#TierObserver` -> `type TierObserver struct`
- `plan:internal/gatecli#RunGate` -> `func RunGate(ctx context.Context, dir, gate string, out, stderr io.Writer, observer TierObserver) int`

**Extends:**
- `internal/gatecli/doc.go`

**Uses:**
- `internal/gatecli#runTest`
- `internal/gatecli#scopesConfigWith`

**Intent:**
Decision 10's in-process gate: the adopt check runs a gate key through the gate's one function, `runTest`, in held slots, and hears each tier as it starts and ends, so it parses no gate output (PATTERN-gate-slot).

`TierObserver` holds `Started func(tier int)` and `Ended func(record gateslot.RunRecord)`; a nil func is skipped and the zero observer observes nothing, which every CLI call passes.
`testRequest` gains `observer TierObserver`.
`runWithSlots` calls `Started` with the run's tier number once its slots are held, or at once for a run that takes none, before the body runs; `report` calls `Ended` with the record it writes, so a carried tier, whose `reportCarried` goes through `report`, ends with no start; `reportBlocked` calls `Ended` with the blocked record. A guard or vet run reports through the same calls with its record's own flags.

`RunGate` runs `runTest` for the gate key `gate` in `dir`, with the defaults `Command` passes, `--wait` set, and `dir` seeded as the context's working directory through `lymcwd.WithCwd`; it writes the envelope to `out` and the summary lines to `stderr`, and returns the call's exit code.
It adds an in-process entry point, bounded to what `lym gate test --wait --gate <key>` already runs: the caller names only the key and the directory, never targets or tiers.

`TestGateTest_Scopes` gains a subtest that calls `RunGate` over the pair with `finalize` and an observer recording each call, with three steps:
- after editing a file, as the sibling subtests do so no tier carries, `Started` and `Ended` are each called once per tier the key runs, in order, `Ended` with the record of that tier and its exit code, and the return equals the exit code the CLI form gives over the same tree;
- a second call over the unchanged tree carries every tier: `Ended` is called once per tier with a record whose `Carried` is set, and `Started` never;
- over a config whose tier0 sets `shared_build` and whose build command exits 1, `Started` is called once for tier0, whose slots run the build, and `Ended` once with its blocked record, and the return is the build's exit code.

`doc.go` gains a paragraph on `RunGate` and `TierObserver`.

**ImpactSummary:** One new observer field on the request, called at three existing report sites; every CLI call passes the zero observer, so output and exit codes are unchanged.

**Commit:** 12: Run a gate key in-process with a per-tier observer
