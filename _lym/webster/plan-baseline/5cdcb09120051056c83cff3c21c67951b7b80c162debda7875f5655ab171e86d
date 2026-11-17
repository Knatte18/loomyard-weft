# Card 18 — catch-up-engine

**Create:**
- `internal/dobbyengine#ErrOriginUnreachable`
- `internal/dobbyengine#CatchUpDeps`
- `internal/dobbyengine#CatchUpRun`
- `internal/dobbyengine#CatchUpReport`
- `internal/dobbyengine#CatchUp`
- `internal/dobbyengine#TestCatchUp`

**Extends:**
- `internal/dobbyengine/doc.go`

**Intent:** Add the catch-up from origin, the entry point the dobby start, the dobby's sleep, a hand pass and later adopt run (D4).

`CatchUp(deps)` runs D4's steps over seams, so `internal/dobbyengine` imports no fabric or teardown package (Told-Geometry):
1. `PullBolt` fetches Bolt and replays local display commits as the board sync does; then `Register` registers this machine when the registry holds no entry of its id, or checks the entry it holds.
2. For each slug `Runs` lists (the local pairs in the caller's scope), `FetchRun` fetches the pair's branches, and `ReadRun` reads the run from the result as a `CatchUpRun`: `Owner`, `OwnedElsewhere` (through the registry) and `Ahead` (local commits on top of origin's tip on either side).
3. A run owned here and `Ahead` gets `PushOwed`, which pushes records first and hands over itself when origin holds another machine's takeover entry; nothing is reset.
4. A run owned elsewhere, a `landing` past its parent-side merge included, gets `HandOver(slug, owner)`.
5. `ApplyPauseIntents` gets the runs owned here, to write this machine's pause entries from their pause intents.
`CatchUpReport` lists the runs pushed, handed over and owned here; the caller reads every listed run as touched.

An unreachable origin at the Bolt fetch or a pair fetch returns an error wrapping `ErrOriginUnreachable` before any step 3 to 5 act, so the catch-up acts on no run.
A registration refusal is returned as `Register` gave it, before any run is read.
A failed push or hand-over of one run is joined into the returned error and the other runs go on; the caller keeps the catch-up owed.
A nil seam is a wiring error returned at once.

`TestCatchUp`, tier0 over fakes, covers: an unreachable Bolt fetch acts on no run; a refused registration stops before the runs; an owned-here run ahead is pushed and one level is not; an owned-elsewhere run is handed over naming its owner, a `landing` run past its parent-side merge included; an unreachable pair fetch stops before any push; a failed hand-over goes on to the next run and is returned; the pause intents go to the owned-here runs only; a hand pass's scope of one slug fetches only that slug.
`doc.go` describes the catch-up and its steps.

**ImpactSummary:** New entry point with its own seams; nothing calls it until cards 20 to 22 wire it.
