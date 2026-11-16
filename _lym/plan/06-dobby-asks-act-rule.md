# Card 6 — dobby-asks-act-rule

**Edit:**
- `internal/battencli#DobbyRuns`
- `internal/dobbyengine#RunView`
- `internal/dobbyengine#planPass`
- `internal/dobbyengine#daemon.drivePasses`
- `internal/dobbyengine#runLabel`
- `internal/dobbyengine#TestPlanPass`
- `internal/dobbyengine#TestDrivePasses_ExitHandling`
- `internal/dobbyengine/doc.go`

**Uses:**
- `plan:internal/runfsm#Read`
- `plan:internal/runfsm#Act`
- `plan:internal/runfsm#OwnerOf`
- `internal/battencli#DobbyParents.Of`
- `internal/dobbyengine#daemon.ownedElsewhere`

**Intent:** The dobby's pass asks the act rule before it gives a run a pass (D1), as the first importer of `internal/runfsm`.

`RunView` gains `Wait bool`, `HandOver bool` and `ActReason string`, the act rule's answer for the run and its cause with its way forward.
`DobbyRuns` fills them for each run: `runfsm.Read` over the prime, then `runfsm.Act` with
`OwnerOf(geom.BoardDir, slug, parents.Of)` for the owner, `Covered` from the run's pause coverage, the later parts' inputs left zero,
and the shed-pause aside set only when the run's work reached its parent, the one paused run today's planner drives (see Shared Decisions).
A read or act failure of one run is that run's `ActReason` with `Wait` set, logged at Warn, and never fails the listing.

`planPass` returns no pass for a run whose `Wait` or `HandOver` is set, before every other rule.
That drops today's covered continue pass: a covered run waits, and gets its full pass at the lift as before.
`drivePasses` runs `daemon.ownedElsewhere` for a `HandOver` run that is not yet recorded owned elsewhere,
the same handling a pass that exits with `PassExitOwnedElsewhere` gets, so the owner pause still lands on this machine.
`runLabel` adds the reason to the label of a waiting or handed-over run, so the dobby's log names the cause and its way forward once per change.
`doc.go`'s "The passes" section states the act rule's place before the trigger rules, the dropped covered continue pass and the hand-over handling.

Tests, tier0: `TestPlanPass` gains rows where `Wait` and `HandOver` give no pass under every trigger, and a covered run gets no continue pass;
`TestDrivePasses_ExitHandling` gains a case where a `HandOver` run gives no spawn and records the run owned elsewhere once.

**ImpactSummary:** One new reader in battencli's dobby view and one early return in dobby's planner; the covered continue pass goes, the full pass at a lift stays.

**Commit:** 6: dobby-asks-act-rule
