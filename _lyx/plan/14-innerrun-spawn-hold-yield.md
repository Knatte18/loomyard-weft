# Card 14 — innerrun-spawn-hold-yield

**Edit:**
- `internal/battenshed#innerRunProducer.holdSpawnWhileCovered`
- `internal/battenshed#innerRunProducer.Call`
- `internal/battenshed#TestInnerRun_OwnerCheck`

**Create:**
- `plan:internal/battenshed#TestInnerRun_SpawnHoldYields` -> `func TestInnerRun_SpawnHoldYields(t *testing.T)`

**Uses:**
- `plan:internal/shedengine#Yield`

**Intent:**
Make the spawn hold yield instead of sleeping (D4): a pass at Run-Shed under a covering pause or a standing owner ends at once with the hold reason.
`holdSpawnWhileCovered` reads coverage once: while a pause covers the slug it writes `held by the pause of <scope> since <paused_at>` to the reason file as today and returns a `Yield` end; while an owner stands and no pause covers, it first runs the owner check when a notice probe is due by the persisted `lastProbe` (recording the probe), and an owned-here verdict clears `ownedBy` and returns nil so `Call` goes on to the spawn in the same pass, while any other verdict writes the owner hold reason and yields.
Batten's own pause flag still returns its pause-requested Stuck, and a cancelled context its error.
The hold no longer sleeps, so it has no time bound of its own and the pass stays in flight only through the in-pass waits D4 names.
`Call` returns the hold's end as its result, so the yield, its reason and the memory record's save go out together.
`TestInnerRun_SpawnHoldYields` (tier0, parallel) rows: a covered absent-status child yields at once with the pause reason in the reason file and no spawn; a covering pause lifted between two Calls: the first Call spawns nothing and yields, and the second, its `Pause` answering uncovered, spawns once.
`TestInnerRun_OwnerCheck` gains rows: with an owner standing and no probe due the pass yields with the owner reason and runs no check; with a probe due and an owned-here verdict the same pass spawns; with an owned-elsewhere verdict it yields and spawns nothing.

**ImpactSummary:** One producer helper stops sleeping and its one caller returns its end; no seam changes.
