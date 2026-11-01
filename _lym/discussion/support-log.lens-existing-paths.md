```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every code path that already writes, lifts or reads a pause entry, plus the passes and resumes the hold now sits in front of.

**Findings**

- **MEDIUM, D4, race between the lift and the first pass.** `lym beam go` lifts the hold and then calls `battencli.StartFirstPass`, but the lift alone already triggers a pass.
  - The live dobby gives a covered run a full pass at the lift (`planPass` doc in `internal/dobbyengine/passes.go`, plus the `wasCovered` transition), so the two race.
  - The record says the driver lock refuses the second driver, but the loser's envelope may come back as `ok: false` or as a failed detached pass.
  - D4 counts "a resume failure is that run's error", so `lym beam go` can report a false error for a run that is in fact running.
  - The record should say the loser is no error, or let dobby's pass at the lift be the resume and keep only the Finalize-halted child's `lym loom resume`.
- **MEDIUM, D3 and D5, adopt's existing standing-pause text.**
  - `internal/adoptcli/adopt.go:752` writes `PauseStanding` naming `lym tender resume &lt;slug&gt;` for any entry of the run's own scope; under D3 that entry is now the hold, so the text would point the operator past `lym beam go` and skip the Finalize-child resume.
  - The step at `adopt.go:735` lifts only `OwnerCaller` entries. It survives D3 as long as the conversion runs before it, but the record should say that the outcome text for a held run names `lym beam go`.
- **MEDIUM, D3 bound, `lym tender resume` releases only part of what `lym beam go` does.** The record names `lym tender resume` as a second release, and that survives as a lift. But no path then does what `lym beam go` does after it: a child halted at Finalize's owner check stays halted. Dobby passes the batten run, while the halted child is resumed only by `lym loom resume`. Through that route the run would read `running_here` while it stands still. The record should state this bound, or have the report read a child halted at Finalize's owner check as held or halted.
- **LOW, D3, turning the owner entry into the hold.** Only `pauserecord.Update` rewrites an entry. Lifts find an entry by scope, caller and pause time (`tenderengine/resume.go:144`), so a ref someone read earlier with `Caller: owner` no longer matches.
  - Dobby's `liftOwnerPause` rereads the record each time, so the hold survives it.
  - Hibernate wake's `holdRuns` adds none when a slug entry already stands, so the hold survives it.
  - `cleanupcli`'s `wakeEntry` matches only `hibernateengine.PauseCaller`, so the hold survives it.
  - A push child exempt under `OwnerEntryExempt` loses its exemption once the caller changes, because `pauserecord/hold.go:293` exempts `OwnerCaller` only. The child waits instead of pushing; the record is silent on this, and it costs only a delay.
- **LOW, D3, the network gate also holds the pair's pushes.** A hold is an ordinary slug entry, so the network gate holds every network call of a held run's processes, and `tenderengine.Lift` spawns the deferred pushes at release. That matches the intent; the record could add one line saying a held pair pushes nothing until `lym beam go`.

**Paths traced, by decision**

- **D3:**
  - Dobby `planPass`, a covered run gets no full pass: survives.
  - Dobby run notice, a covered child is in no condition: survives.
  - Dobby `liftOwnerPause` (`beamcli/dobbydeps.go:315`), lifts owner entries only: survives.
  - Adopt's owner lift (`adopt.go:735`): survives if it runs after the conversion.
  - Adopt's `PauseStanding` text (`adopt.go:752`): misreads; finding above.
  - Hibernate wake `holdRuns`, which adds no second entry: survives.
  - `lym beam start`'s owner pause of runs owned elsewhere, also one entry per scope: survives.
  - `lym batten cancel`'s `slugPauseRefs` lift: lifts the hold together with the cancel, which is correct.
  - `cleanupcli` `wakeEntry` / `LiftWakeHold`, which match the hibernate caller only: survives.
  - `tender resume &lt;slug&gt;` and bare `tender resume`: lift the hold but skip the Finalize-child resume; finding above.
  - The orch watcher's `settledWithoutInterrupt`: unaffected, since a hold covers no orch scope.
  - The network gate's `OwnerEntryExempt` (`hold.go:293`): an exempt push child loses its exemption after the conversion (LOW).
  - `Update`'s `withoutExpired`, which drops only resumed entries: the hold persists.
  - The origin reconcile's `hubOnlyPaused` / `LivePair`, which read hub entries only: unaffected.
- **D4:**
  - Dobby's full pass at the lift against `StartFirstPass`: they race; finding above.
  - `tenderengine.Lift` spawning the deferred pushes: survives, as intended.
- **D5:**
  - The batten record row, the pause record and owner reads: they read correctly, except a halted child reached through `tender resume` (MEDIUM above).
- **D2:** `ReconcileOrigin` is unchanged: survives.
- **D6:** Dobby's start of released tasks reads only a pause on the task's own slug or the hub, so with no hold on such a task it starts: survives.

**Tool calls**

- `pauserecord/record.go` `Update` / `withoutExpired`, `hibernateengine/wake.go` `holdRuns`, and a grep for every reader of `.Caller`: my context lacked who else writes or matches entries by caller.
- `cleanupcli/reconcile.go` wake-hold seams, `orchengine/pause.go` `settledWithoutInterrupt`, and `battencli/cancelcmd.go` `slugPauseRefs`: the grep found these as caller readers whose bodies my context lacked.
