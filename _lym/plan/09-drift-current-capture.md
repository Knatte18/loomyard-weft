# Card 9 — drift-current-capture

**Edit:**
- `internal/driftengine#RunCapture`
- `internal/driftengine#runStep`
- `internal/driftengine#stepRun`
- `internal/driftengine#acquireSlot`
- `internal/driftengine#captureInto`
- `internal/driftengine#RunCompare`
- `internal/driftengine#NewPaths`
- `internal/driftengine#TestRunCapture`
- `internal/driftengine/doc.go`
- `internal/loomcli#driftDeps`
- `internal/testkit/envkit#driftDeps`
- `internal/battencli#intentionallyNil`

**Extends:**
- `internal/driftengine#Paths`
- `internal/driftengine#Deps`
- `internal/driftengine#CaptureRequest`

**Create:**
- `plan:internal/driftengine#CurrentCapture` -> `type CurrentCapture struct`
- `plan:internal/driftengine#CaptureCurrent` -> `func CaptureCurrent(ctx context.Context, req CaptureRequest) (CurrentCapture, error)`
- `plan:internal/driftengine#TestCaptureCurrent` -> `func TestCaptureCurrent(t *testing.T)`

**Uses:**
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/gateslot#InheritEnv`
- `internal/gateslot#Pool.Inherited`
- `internal/loomcli#driftCaptureRequest`

**Intent:** D7's capture order and the current capture every check runs.

- Slot before lock: `runStep` no longer acquires a slot itself; `stepRun` gains the held slot, and `acquireSlot` runs before the capture lock in `RunCapture` and `CaptureCurrent`.
  `CaptureRequest` gains `InheritedSlot`, the slot a caller running inside a gate holds: the wiring reads it from the process environment's `gateslot.InheritEnv` when it builds the request, and the engine never reads the environment.
  When `InheritedSlot` names a slot `Pool.Inherited` reports held, the capture runs in that slot and acquires none, as a command inside a gate does (`PATTERN-gate-slot`); otherwise it waits for one slot as today.
  `RunCompare` and `captureInto` take their slot through the same split, keeping their order; the main capture still refuses a lock held past `LockRetry`, with the same refusal.
- `Paths` gains `CurrentDir`, `.lym/drift/current`, declared in `NewPaths`.
- `CaptureCurrent` creates a fresh directory under `CurrentDir`, named by the UTC stamp and the pid so concurrent checks never share one, takes its slot, then waits for the capture lock with no bound of its own, sleeping and retrying while it is held, since a lock holder already holds its slot (D7).
  It runs the declared capture command with `{baseline_dir}` rendered as that directory, writes no `capture.json`, never touches `BaselineDir` or its backups, and releases the lock and the slot on return.
  `CurrentCapture` holds `Dir`, `Log`, `Passed` (exit 0 within the timeout and at least one file written) and `Reason` (the failure, the directory the command ran in and its last output, as a failed main capture's reason reads).
  A step refusal, such as `missingDirRefusal` for a dir that is no directory, is `Passed` false with the refusal, which names its way forward, as `Reason`, never a returned error, so every check maps it to a capture fault the stop chain carries; a returned error is an I/O failure only.
- `Deps` gains `CaptureCurrent func(ctx context.Context) (CurrentCapture, error)`; `loomcli`'s `driftDeps` wires it over `driftCaptureRequest` with the step from `StepWithHubEnv` and `InheritedSlot` from the environment, `envkit`'s `driftDeps` fills a passing fake, and `battencli`'s `intentionallyNil` lists it.

`TestRunCapture` gains a row: with the pool's only slot taken and the lock free, the main capture holds no lock while it waits for the slot.
`TestCaptureCurrent` (tier1, real shell command, `t.Parallel`) covers: two concurrent current captures each write their own directory, neither writes `capture.json`, and the baseline directory and its backups are untouched; a current capture finding the lock held waits and finishes once it is released; with a pool of one slot held by a gate, a current capture told that slot as `InheritedSlot` and a second one told none both finish, the second once the first released the lock and the gate its slot (D7's "a check inside a gate and a fork's check with one free slot both finish"); a failing command is `Passed` false with its reason and log; a command that exits 0 and writes no file is `Passed` false naming the directory; a step whose dir is no directory is `Passed` false with the refusal naming `lym config drift`.
The test sets `InheritedSlot` on the request, so it touches no process environment and stays parallel.
The doc.go's `# Capture` section states the order and the current capture.

**ImpactSummary:** Reorders slot and lock inside the package's capture and compare runners, with the main capture's refusal kept; adds one seam wired in loomcli and faked in envkit.
