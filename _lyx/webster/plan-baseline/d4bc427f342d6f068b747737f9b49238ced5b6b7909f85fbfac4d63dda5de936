# Card 7 — land-weft-stop

**Create:**
- `plan:internal/battencli#pullStopOutcome` -> `func pullStopOutcome(err error) (recordsStop string, pullErr error)`
- `plan:internal/battencli#TestPullStopOutcome` -> `func TestPullStopOutcome(t *testing.T)`

**Edit:**
- `internal/battenshed#LandDeps`
- `internal/battenshed#landProducer.pullAndDeploy` -> `func (p *landProducer) pullAndDeploy(ctx context.Context, parent string) (step, reason, weftStop string, err error)`
- `internal/battenshed#rowNotifier.ready` -> `func (n rowNotifier) ready(ctx context.Context, landed string, slugs []string, weftStop string)`
- `internal/battenshed#landProducer.Call`
- `internal/battencli#landDeps`
- `internal/battenshed#TestLand_ReadyNotice`
- `internal/battenshed#landFixture.deps`
- `internal/battencli#newFakeReceiver`
- `internal/shedrecipe#newTestEnv`
- `internal/testkit/envkit#FullEnv`

**Uses:**
- `plan:internal/fabricengine#WeftStopError`

**Intent:** D9's last clause: batten's Land row, which pulls the prime after a landing, deploys after a weft stop as it does today and names the stop in its landing notice to the orch.
`LandDeps.Pull`'s field type becomes `func(ctx context.Context) (weftStop string, err error)`: `battencli#landDeps`' closure returns the two results of `pullStopOutcome` over the pull's error.
`pullStopOutcome` returns a stop message with a nil error only when `err` is exactly a `*fabricengine.WeftStopError`, by a direct type assertion, with no warp failure joined to it, and returns any joined or other error as `pullErr`: card 6's `Pull` joins the weft stop and the warp partial error (`TestPull_WeftDivergedAndWarpFetchFails_PartialError`), and an `errors.As` over such a join would find the stop and drop the warp error, so a warp pull failure stays the Land row's `pull` Stuck and the deploy never runs over an unpulled prime.
`pullAndDeploy` passes `weftStop` back and still runs the deploy after it; `landProducer.Call` hands it to `rowNotifier.ready`, which appends `; the prime's records did not pull: <stop>` to the landing notice when it is non-empty.
A pull error other than a weft stop stays the `pull` step's Stuck, unchanged.

`TestLand_ReadyNotice` gains a row whose fake pull reports a weft stop: the deploy runs and the one landing notice names the stop.
`TestPullStopOutcome` (untagged, table-driven) has rows for a lone weft stop (its message, nil error), a weft stop joined with a warp error (no stop, the joined error) and an unrelated error (no stop, that error).
The four test `LandDeps` literals take the new `Pull` type, returning an empty stop, since each would otherwise stop compiling: `landFixture.deps`, which every battenshed Land test builds through, `internal/battencli`'s `newFakeReceiver`, `internal/shedrecipe`'s `newTestEnv` and `internal/testkit/envkit`'s `FullEnv`, which every test binary importing `envkit` builds.

**ImpactSummary:** One struct field re-typed and two unexported methods re-signed inside battenshed, with one caller each; battencli's Land wiring is the only production filler, through the new `pullStopOutcome`, and the test literals above are the only other fillers.

**Commit:** 7: batten: Land deploys after a weft stop and names it
