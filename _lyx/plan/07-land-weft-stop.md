# Card 7 — land-weft-stop

**Edit:**
- `internal/battenshed#LandDeps`
- `internal/battenshed#landProducer.pullAndDeploy` -> `func (p *landProducer) pullAndDeploy(ctx context.Context, parent string) (step, reason, weftStop string, err error)`
- `internal/battenshed#rowNotifier.ready` -> `func (n rowNotifier) ready(ctx context.Context, landed string, slugs []string, weftStop string)`
- `internal/battenshed#landProducer.Call`
- `internal/battencli#landDeps`
- `internal/battenshed#TestLand_ReadyNotice`
- `internal/battenshed#landFixture.deps`
- `internal/battencli#newFakeReceiver`

**Uses:**
- `plan:internal/fabricengine#WeftStopError`

**Intent:** D9's last clause: batten's Land row, which pulls the prime after a landing, deploys after a weft stop as it does today and names the stop in its landing notice to the orch.
`LandDeps.Pull`'s field type becomes `func(ctx context.Context) (weftStop string, err error)`: `battencli#landDeps`'s closure returns a `*fabricengine.WeftStopError`'s message as `weftStop` with a nil error, and every other pull error as the error, as today.
`pullAndDeploy` passes `weftStop` back and still runs the deploy after it; `landProducer.Call` hands it to `rowNotifier.ready`, which appends `; the prime's records did not pull: <stop>` to the landing notice when it is non-empty.
A pull error other than a weft stop stays the `pull` step's Stuck, unchanged.

`TestLand_ReadyNotice` gains a row whose fake pull reports a weft stop: the deploy runs and the one landing notice names the stop.
The two `LandDeps` literals in the tests take the new `Pull` type, returning an empty stop: `landFixture.deps`, which every battenshed Land test builds through, and `internal/battencli`'s `newFakeReceiver`, which would otherwise stop compiling.

**ImpactSummary:** One struct field re-typed and two unexported methods re-signed inside battenshed, with one caller each; battencli's Land wiring is the only production filler.

**Commit:** 7: batten: Land deploys after a weft stop and names it
