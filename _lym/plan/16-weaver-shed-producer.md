# Card 16 — weaver-shed-producer

**Create:**
- `plan:internal/shedadapters#WeaverRunner` -> `type WeaverRunner func(weaverengine.RunDeps) (weaverengine.RunResult, error)`
- `plan:internal/shedadapters#WeaverProducer` -> `type WeaverProducer struct`
- `plan:internal/shedadapters#NewWeaverProducer` -> `func NewWeaverProducer(name string, run WeaverRunner, deps weaverengine.RunDeps) *WeaverProducer`
- `plan:internal/shedadapters#WeaverProducer.Call` -> `func (p *WeaverProducer) Call(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/shedadapters#claimConflict` -> `func claimConflict(name, engine, reason, conflictDir string, ptr shedengine.OutputPointer) shedengine.OutputPointer`
- `plan:internal/shedadapters#TestWeaverProducer` -> `func TestWeaverProducer(t *testing.T)`

**Edit:**
- `internal/shedadapters#WebsterProducer.Call`
- `//docs/overview.md`

**Extends:**
- `internal/shedadapters/doc.go`

**Uses:**
- `internal/shedadapters#TestWebsterProducer_OutcomeStuck`
- `internal/shedadapters#TestWebsterProducer_ConflictRerouteCapThenRepeatHalts`
- `internal/shedadapters#planConflictWayForward`
- `internal/runplan#ClaimConflictRoute`
- `plan:internal/runplan#ConflictDir`

**Intent:** Map Weaver's run onto the shed producer contract, with the same Done, reroute and Stuck routes as Webster's (D1, D2, D6).

- `claimConflict` is the plan-conflict claim `WebsterProducer.Call`'s stuck branch holds today, lifted out implementer-neutral: it claims the record in the conflict directory and returns the Stuck pointer, `Reroute` set on a new conflict, the recurrence reason on a repeat, and the plain reason when none or the claim fails (logged).
  Its reason texts read `<engine> plan conflict`, so Webster's read as they do today, `planConflictWayForward` keeps its text, and the Webster tests that pin them stay unchanged.
  `WebsterProducer.Call` calls it.
- `WeaverProducer.Call` checks the context, runs `run` (default `weaverengine.Run`) with a `ReentryStep` naming the row, and maps:
  Done to Done pointing at Weaver's `summary.md`; a Stuck result through `claimConflict`; `ErrRunBusy` and a fault in Weaver's config or state (an error wrapping `ErrActiveRefused`, a wiring error or a state read or write error) to Stuck with the error's text, which names its way forward; a cancellation to the context error, as Webster's adapter does.
- `docs/overview.md`'s `internal/shedadapters/` line names Weaver's adapter beside Webster's.

`TestWeaverProducer` is tier0: Done points at the summary; a stuck run with a new conflict record reroutes and a repeat does not; a stuck reason passes through; a busy run and a state fault are each Stuck with their text; a cancelled context returns the context error.
The rows of `TestWebsterProducer_OutcomeStuck` and `TestWebsterProducer_ConflictRerouteCapThenRepeatHalts` keep covering Webster through the lifted claim.

**ImpactSummary:** One shedadapters function's stuck branch moves into a shared helper with the same outcomes and texts; Webster's tests stay as they are.
