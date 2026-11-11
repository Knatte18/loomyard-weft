```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: tracing what the record's changed state meets in code that already runs.

**Findings**

1. **MEDIUM, D1 and D13 (re-stamp against `testmap-cross-package`'s format marker).** A D1 re-stamp moves the map's `Commit` but says nothing about the marker's own `Commit` and `BuiltAt`.
   - `coverage-test-select`'s D5 makes a map a miss when its marker's `Commit` and `BuiltAt` differ from the map's own. That check exists to catch a map an older binary rewrote.
   - If the re-stamp moves `Commit` but not the marker, every re-stamped map becomes a gate miss. That undoes D1's purpose.
   - Evidence: `internal/testcover/coveragemap.go` stores `Commit` and `BuiltAt` as plain fields, and the T14 record's D5 text spells out the marker-mismatch miss.
   - Fix: D1 should say a re-stamp, a roll-forward and a D5 failure record each rewrite the marker's `Commit` and `BuiltAt` together with the map's. It should also say whether `BuiltAt` moves on a re-stamp.

2. **MEDIUM, D5 (redundancy step reads `BuildFailure`).** `internal/gatecli/redundancy.go` (around line 305) refuses to judge any package whose map has a non-empty `BuildFailure`, reporting "its coverage map records a build failure".
   - D5 keeps the old entries and records the failure in the map. If that reuses `BuildFailure`, the redundancy step drops a usable map.
   - The record covers the selector's reading of a failure-with-entries map (D2) but not this reader.
   - Fix: D5 should say how the redundancy step reads a failure-with-entries map. It should judge by the kept entries, consistent with D2's selector rule.

3. **MEDIUM, D8 and D12 (hand calls overwrite the exit record).** Today `close` in `internal/gatecli/testmap.go` writes `testmap-exit.json` on every call, hand calls included.
   - D8's trigger reads the planned HEAD and the queued count from the last exit record.
   - A hand call naming packages plans nothing. If it writes the record, it can blank the queue, which loses the backlog until the next landing. It can also stamp a planned HEAD it never planned, so no re-plan follows a landing.
   - Fix: D8 or D12 should say only an upkeep pass writes those fields, and a hand call leaves them as they were.

4. **MEDIUM, D8 and D9 (an orphaned or waiting worker is unbounded).**
   - The dobby doc's bound says a dobby that dies without a stop leaves the refresh running "until the verb ends within its own timeouts". Today `--max` gives that bound.
   - Under D8 and D9 a worker can wait without limit for a polite slot while its queue is non-empty, and it reads HEAD only before a package job.
   - So a worker parked in D9's wait never notices a landing, and an orphaned one never ends.
   - D8's own bound, "a landing ends its work at the next package job", does not hold while the worker is waiting.
   - Fix: D9's wait should also re-read HEAD and end when it moved. The record should state the orphan bound.

5. **LOW, D9 (inherited slot).** When `gateslot.InheritEnv` (`LYM_GATE_SLOT`) names an enclosing run's slot, `holdSlot` skips its acquire. Drift steps and gate runs export that variable to their children.
   - D9's yield, which releases and later re-takes the slot, has no meaning for a slot the call does not own.
   - D9 covers hand calls but not a call that runs under an inherited slot.
   - Fix: say that a call running under an inherited slot neither takes a slot nor yields.

6. **LOW, D8 (failed registration).** The dobby doc says a registration that fails leaves the refresh unregistered, so `TestMapRunning` reads false.
   - Today the next start is bounded by `--max`. Under D8 the uncapped worker lets a second worker start beside an unregistered first one, and both hold polite slots.
   - The build lock keeps the two off the same package, so this is a waste, not a wrong result. The bound is worth one line.

7. **LOW, D9 and D13 (hand call skips a locked package).**
   - `lockPackage` takes the per-package build lock without waiting, and skips the package when another call holds it.
   - So the hand call `redundancy-sweep-existing` relies on (D13) silently skips a package that upkeep is measuring at that moment.
   - The record says the hand call "rebuilds the named packages whole", which is not true while upkeep holds that package.

**Paths traced: survives or breaks**

| Decision | Path | Result |
|---|---|---|
| D1 | the gate's map read (T15 D8) during a re-stamp | survives: `WriteMap` writes atomically, and either commit is safe |
| D1 | `mapDrift` (redundancy) compares `Files` at the merge base | survives |
| D1 | redundancy `MapAge` read from `BuiltAt` | survives, but the age it reports is ambiguous after a re-stamp (finding 1) |
| D1 | T14 D5's marker check | breaks (finding 1) |
| D4 | `judgeRuns` cover set | survives if the build adds the outcome-entry filter D4's criterion names; today a spawn-marked map test counts as a cover, which D4 changes on purpose |
| D5 | redundancy `reasonMapFailure` | breaks (finding 2) |
| D5 | `stale`'s once-per-HEAD failure rule | survives: D1 replaces it |
| D7 | `gateslot/config.go`, which reads, validates and maps `test_map_max_age_days` | survives with the retired-key entry; the module already has a `RetiredKeys` map |
| D7 | `testmap.go`'s `maxAge` | removed with the key: survives |
| D8 | `dobbyengine/tidy.go` adding to `LandingsSinceTestMap` | survives: the field becomes write-only until it is removed, and a tolerant record keeps it readable |
| D8 | `dobbyengine/testmap.go` trigger | replaced: survives |
| D8 | exit record written by a hand call | breaks (finding 3) |
| D8 | dobby death leaving an orphan worker | breaks the bound (finding 4) |
| D8 | failed registration | weakened (finding 6) |
| D9 | `InheritEnv` inherited slot | unaddressed (finding 5) |
| D9 | build lock against a hand call | partial (finding 7) |
| D10 and D11 | `orchcli/cli.go`'s `EndRole(RoleTestMap)` | survives: D10's signal handling writes the finished rounds |
| D10 and D11 | `lym beam stop`, `BeamDown` | survives: D10's signal handling writes the finished rounds |
| D11 | anchor-move wait | replaced by `EndRole`: survives |
| D12 | `ReadExitRecord` | survives: it has no production reader today, and the dobby becomes its first |

**Tool calls**

1. A grep for the readers and writers of `ReadMap`, `BuiltAt`, `BuildFailure`, `ReadExitRecord`, `LandingsSinceTestMap`, `TestMapMaxAgeDays` and `RoleTestMap`. My context named those symbols but not who else reads them.
2. A grep for the `InheritEnv` callers, the anchor-move wait site and the map's `Commit` and `BuiltAt` fields. My context held `holdSlot`'s inherit skip only from a summary.
3. A read of `internal/dobbyengine/doc.go`'s test-map bounds and a grep for other dobby users of the refresh. My context had a summary of those bounds, not their text.
