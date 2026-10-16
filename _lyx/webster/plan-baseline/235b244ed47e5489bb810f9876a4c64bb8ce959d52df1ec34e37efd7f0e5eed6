# Card 10 — startup-gate-seam

**Edit:**
- `internal/shuttleengine#TestProviderSeamImportRule`
- `internal/shuttleengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/shuttleengine#Runner.AcceptStartupGate`
- `internal/shuttleengine#TestRunner_AcceptStartupGate`

**Uses:**
- `internal/shuttleengine#Runner.interruptEngine`
- `internal/shuttleengine#liveStrand`
- `internal/shuttleengine#playInputsSent`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#newFixture`
- `internal/shuttleengine#withStrand`
- `internal/shuttleengine#withEngines`
- `internal/shuttleengine#liveStrandStatus`
- `internal/shuttleengine#undismissableEngine`

**Intent:** The shuttle half of #678 (D9): an engine-neutral seam that accepts a provider's startup gate on a live strand.

`Runner.AcceptStartupGate` resolves the engine of the strand with the guid the way `Runner.interruptEngine` does, the run record's engine or, for a hand-added strand with no record, the live strand's provider, and fails when the strand is not live or its run record cannot be read.
It captures the pane once and runs the engine's own `Startup` classification on it; only on `StartupTrustPrompt` does it play the engine's `TrustDismissSequence` for that capture through `playInputsSent`, logging the acceptance at Info, and it reports whether it sent any input.
A ready or pending capture, and a gate whose sequence is empty because the engine cannot locate its accepting option, send nothing and report false; a capture error and a send error are returned.
The trust-gate needles and accept keys stay under `internal/shuttleengine/claudeengine` (`PATTERN-shuttle-provider-seam`); the seam calls only the `Engine` methods.
Bound: it plays only the provider's own gate-accept sequence, only on the strand the caller names and only when the provider's classification reads a gate; for claude that covers the folder-trust gate and the bypass-permissions risk gate, and claude shows the latter only to a strand whose launch flags or settings already ask for bypass permissions, so the seam grants no permission the strand did not request.
Accepting the folder-trust gate adds the strand's working directory to the operator's Claude trust list.
Its only caller is the llm smoke test's helper; a production caller is a new decision.
`TestProviderSeamImportRule`, the package's seam-enforcement scan, pins that through a call-site scan beside its import scan: no production `.go` file outside `internal/shuttleengine/` contains `.AcceptStartupGate(`, with a planted-file row in another package proving the scan finds a call and one in `internal/shuttleengine/` proving it is let through, and a `RealModule` check over the module.
Since the test reads repository files outside its own package, it gains `//lyx:guard` stacked with its `//testtiming:keep` above its func line (`PATTERN-test-speed`), and the keep reason names the call-site scan too.

Test `TestRunner_AcceptStartupGate`, over the `fakeReed` and `fakeEngine` fixture with a live strand:
- a capture the engine classifies `StartupTrustPrompt` sends the engine's sequence, one Enter, reports true, and hands `TrustDismissSequence` that same capture;
- a capture classified `StartupReady`, and one classified `StartupPending`, each send no key and report false;
- a gate capture over the existing `undismissableEngine`, whose sequence is empty, sends no key and reports false;
- a strand with no run record, in a set built with `withEngines` whose default is one `fakeEngine` and whose second `fakeEngine` the live strand's provider names, accepts the gate through the second engine: only its `TrustDismissCaptures` holds the capture;
- a run record that cannot be read returns its error and sends nothing;
- a dead strand returns an error and sends nothing;
- a failing capture returns its error and sends nothing.

`cmd/lyx/testdata/test-budget.yaml` raises the `internal/shuttleengine` row by the one new top-level test.

Docs in the same commit: `internal/shuttleengine/doc.go` documents `Runner.AcceptStartupGate` beside the startup-gate dismissal Wait does, with its bound, its one caller and the scan that pins it.

**ImpactSummary:** Adds one exported `Runner` method built from existing engine calls and helpers in `internal/shuttleengine`; no production member changes, and the seam-enforcement test, the budget row and the package doc are the only other edits.
