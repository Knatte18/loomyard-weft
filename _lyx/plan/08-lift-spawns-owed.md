# Card 8 — Every lift spawns the pushes and the board sync a pause deferred

**Create:**
- `internal/fabricengine#Bolt.HasUncommitted`
- `internal/tendercli#syncBoardIfOwed`
- `internal/tendercli#TestSyncBoardIfOwed`

**Edit:**
- `internal/tenderengine#Deps`
- `internal/tenderengine#ResumeReport`
- `internal/tenderengine#liftEntries`
- `internal/tenderengine#TestResume`
- `internal/tendercli#HubDeps`
- `internal/tendercli#resumeCmd`
- `internal/tenderengine/doc.go`
- `internal/tendercli/doc.go`
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/tenderengine#fakePair`
- `internal/fabricengine#SpawnPairPush`
- `internal/fabricengine#SpawnStatusPush`
- `internal/fabricengine#Bolt.HasUnpushed`
- `internal/boardengine#SpawnSync`

**Intent:**
D6's lift half: a deferred push or sync left its commits and flags local, so the lift that uncovers them spawns what they owe.
`liftEntries` is shared by `Resume` and `Lift`, which `lyx tender resume`, the hibernate wake and cancel, `lyx adopt` and the cleanup reconcile all reach, so the spawn lives there once.

`Deps` gains three seams, each nil meaning nothing is spawned: `PushPair func(slug string) error`, `PushPrimeRecords func() error` and `SyncBoardIfOwed func() error`.
`ResumeReport` gains `Spawned []string` (`spawned`: `pair:<slug>`, `prime-records`, `board-sync`) and `SpawnFailed []SessionNote` (`spawn_failed`), both empty lists when nothing was lifted.
After the record update, and only when it lifted at least one entry, `liftEntries` collects the pairs the lifted entries covered: every pair of `deps.Pairs` for a lifted hub entry, the pair of that slug for a lifted slug entry; it drops each pair a live entry of the updated record still covers and each pair `deps.Owners` reads owned by another machine.
A pair with no `deps.Pairs` member was removed, and its owed push goes with it.
An `Owners` error spawns no pair push and records the error under `SpawnFailed`, since an owner it cannot read may be another machine.
It then calls `PushPair` once per remaining pair, `PushPrimeRecords` once and `SyncBoardIfOwed` once; a seam error is recorded under `SpawnFailed` and never fails the lift.
The resume lines are sent as before; the spawns run after them.

The board sync D6 names spawns "when Bolt is ahead of its upstream"; a sync deferred at its start (card 7) exits before it commits, since board writes only touch the filesystem and the sync is what commits them, so the change it owes sits uncommitted, not ahead.
`syncBoardIfOwed` therefore spawns when Bolt is ahead or holds uncommitted changes, the reading that carries D6's "spawns what they owe" to a deferred sync; the overview's Shared Decisions record this reading of D6.
`Bolt.HasUncommitted` reports whether the board worktree has changed files, through `gitrepo`'s `WorktreeChangedFiles`, inside `fabricengine` (`PATTERN-fabric-git`).
`HubDeps` fills the seams: `PushPair` calls `fabricengine.SpawnPairPush` over the pair's resolved location, `PushPrimeRecords` calls `fabricengine.SpawnStatusPush(prime, "")`, and `SyncBoardIfOwed` calls `syncBoardIfOwed` over the board's `Bolt.HasUnpushed`, `Bolt.HasUncommitted` and `boardengine.SpawnSync(BoardDir)`.

`TestResume` gains rows over the existing fake pairs and recording seams: a lifted slug entry spawns that pair's push, the prime records push and the board sync; a lifted hub entry spawns every pair's push but one a remaining slug entry still covers and one `Owners` reads owned elsewhere; a lifted entry whose pair is gone spawns no pair push; a resume that lifts nothing spawns nothing; a failing `PushPair` is listed under `spawn_failed` and the lift still succeeds; an `Owners` error spawns no pair push.
`TestSyncBoardIfOwed` rows: ahead spawns; level with uncommitted changes spawns; level and clean spawns nothing; a probe error is returned with nothing spawned.
The `# Resume` section of the `tenderengine` doc, `resumeCmd`'s help and the `tendercli` doc state the spawn, its exclusions and the two report keys; the `fabricengine` doc names `Bolt.HasUncommitted`.

**ImpactSummary:** three nil-safe Deps seams, two report fields and one Bolt probe; Resume, Lift and every caller keep their signatures, and only HubDeps fills the seams.

**Commit:** 8: every lift spawns the pushes and the board sync a pause deferred
