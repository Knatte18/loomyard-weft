# Card 34 — dobby-owned-elsewhere

**Edit:**
- `plan:internal/dobbyengine#Deps`
- `plan:internal/dobbyengine#daemon.passExited`
- `plan:internal/dobbyengine#planPass`
- `internal/dobbyengine/doc.go`

**Create:**
- `plan:internal/dobbyengine#daemon.ownedElsewhere` -> `func (d *daemon) ownedElsewhere(ctx context.Context, slug string, now time.Time)`
- `plan:internal/dobbyengine#daemon.recheckOwnerPause` -> `func (d *daemon) recheckOwnerPause(ctx context.Context, slug string)`
- `plan:internal/dobbyengine#TestOwnedElsewhere` -> `func TestOwnedElsewhere(t *testing.T)`

**Intent:**
Handle a run a full pass found owned by another machine (D4, D7).
`Deps` gains the owner pause (`tendercli.PauseOwnedElsewhere` in production), the release of the parent lock record naming the run's child (the same release InnerRun's `ownedElsewhere` makes), the pull of the Bolt level with origin, the effective owner of the slug on the local Bolt, the lift of a pause entry of the slug's own scope carrying `pauserecord.OwnerCaller`, and the read of whether a live pause entry covers the slug.
On `PassExitOwnedElsewhere` from a run with a status file, `ownedElsewhere` keeps the run's full pass owed, records it owned elsewhere in the state record, pauses it under the owner, releases its parent lock and posts the owner notice naming `lyx adopt <slug>`, once per owner episode by the state record's owner-noticed mark; it posts no failing-pass notice.
A failed owner pause warns once per episode, records the pause failed, and is never retried blind: while the mark stands and no pause entry covers the slug, `planPass` gives the run a full pass on each probe tick, which is refused as owned elsewhere again (and the handling runs again, its pause included, the marker keeping the notice to one) or exits 0 and clears the mark.
While the mark stands and a pause entry covers the slug, `planPass` gives no pass, a dobby start's and a wake's included, but the full pass at the lift of the covering pause, which `lyx adopt` reaches since adopt lifts the owner pause.
Once its owner pause stands, `recheckOwnerPause` brings the local Bolt level with origin on each probe tick, through the same pull a batten start makes, and then reads the slug's effective owner from it; when it no longer reads another machine it lifts its own owner pause entry and runs a full pass, and an unreachable origin or a failed pull lifts nothing that tick, so a stale local Bolt never lifts a pause the pass would put back, so an adopt landing in that window strands no run on a manual step (Added after Discussion, 2026-10-25).
A full pass of a marked run clears the mark when it exits 0, and a further owned-elsewhere refusal leaves it standing.
A run so marked is not in flight for the start count and is in flight for the backoff only while no pause entry covers its slug.
An owner InnerRun's own check finds inside a continue pass sets no mark: InnerRun pauses the run itself and dobby goes on passing it in continue passes.
The doc states the handling and its bounds.
`TestOwnedElsewhere` (tier0, fakes, parallel) rows: an owned refusal pauses, releases the lock and notices once across two refusals; a failed pause gets a full pass on the next probe tick with no second notice; a covering owner pause stops every pass, a restart's included, until its lift, which runs a full pass whose exit 0 clears the mark; a pulled Bolt that reads this machine again lifts the owner pause and runs a full pass, while a failed pull lifts nothing; a dobby whose only run is marked owned elsewhere under a covering pause passes the idle flag to `nextWait`, and that run uncovered counts as in flight.

**ImpactSummary:** One more exit handled by the engine's scheduler and one per-tick owner re-read; the seams are filled by the beam verb's wiring.
