```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked the decision record's claims and bounds.

**Claims attacked**

- D1, "every lock prober holds the lock only between a try and a release": held (`internal/dobbyengine/start.go:243-255`; `state.json` uses its own `.lock`, `state.go:114`).
- D1, unregistered window "at most wiring time plus `lock.ProbeGrace`" for a fresh start: held.
- D1, "a re-exec keeps the old image's record, keyed on the same pid and start ticks": held.
- D2, "the start ticks keep a fresh dobby that reuses a dead dobby's pid … from reading itself as a re-exec": **broken** (F2).
- D2, bound "either way the hub keeps one dobby": held on dobby count, but the refused-re-exec branch posts a false alarm (F5).
- D3, "a `placeDobby` refusal has no Geometry and no log path": **broken** (F1).
- D3, the wiring failure posts over a worktree resolved from `Geometry.WorktreeRoot`: held.
- D4, "a start rewrites only pid, start time and start ticks; a re-exec spawns no second agent": held for a real re-exec, but rests on F2's pid-only test.
- D5, "no raw process output reaches the file": **broken** for a dobby spawned by a pre-change binary (F3).
- D5, "only the lock holder rotates, so two renames never race": held.
- D5, `dobby.out` bounded at each spawn and growing between spawns only by quiet stderr: held. The logger's stderr sink is silent below its threshold (`internal/logger/logger.go:13`).
- D6, "the heartbeat adds no wake": held (`idleCeiling` is one minute, `loop.go:23`).
- D6, "a dead dobby shows in its log within ten minutes": **broken** by about a minute (F7).
- D8, "a tidy on the landing machine covers it, and an adopt onto another machine never tidies it again": **broken** for state already accumulated (F4).
- D8, untidied landings kept in `LandedSinceTidy` until the next tidy on that machine: held (`tidy.go:101-130`).
- D9 and Open risks, returning `adopt` to the template by setting it through `lym config gate`: held, since the base strip is one-time.
- D10, the lock beside `check.json` serializes two checks and the loser writes nothing: held.
- D10, bound "that envelope's pid then names a check that never ran" (the second adopt's): **broken** in direction (F6).
- D11, the dead-check test on pid plus start ticks: held.

**Findings**

- **F1, MEDIUM, design, Decision 3, second bullet ("A `placeDobby` refusal has no Geometry and no log path").**
  The premise is false for every refusal after `lymcwd.ResolveWorktree` succeeds, which is most of them: `RequireDrivableWorktree`, `PrimeName`, `ReadOrigin`, "neither the prime nor a beam", the prime resolve and `DobbyGeometry` (`internal/beamcli/cli.go:411-433`).
  Each of those holds a resolved location. The worktree's `.lym/beam` directory, holding `dobby.log` and `state.json` for the re-exec test, derives from it the way `hubgeom/dobbygeom.go:47` builds `beamDir`. The location is also the worktree `NotifyFromScoped` needs.
  Only a `filepath.Abs` or `ResolveWorktree` failure truly lacks a log path.
  Counterexample: a deploy whose new image refuses the prime at `RequireDrivableWorktree` or `PrimeName` after a re-exec ends every dobby with only an envelope in `dobby.out`. That is no `dobby.log` line and no post, the #880 shape the entry's Expected section forbids.
- **F2, MEDIUM, design, Decisions 2 and 4.**
  The start-ticks guard covers only the lock wait.
  `recordStart` (`internal/dobbyengine/loop.go`, the `found && record.PID == os.Getpid()` early return) decides start or re-exec by pid alone and writes nothing on a match.
  Counterexample: after a reboot a fresh dobby reuses the dead dobby's pid. It waits only `ProbeGrace` (D2), but `recordStart` reads it as a re-exec: it sets no `owesAll` and never writes its own start ticks.
  At its first real re-exec the record still holds the dead dobby's ticks, so the re-exec reads as fresh. It waits only `ProbeGrace`, and a refused start posts nothing: the #880 path comes back.
  The same holds for an old-format record that carries this pid and no ticks.
  The record never says that `recordStart`'s test and the rewrite D4 describes use the start ticks too.
- **F3, LOW, design, Decision 5 ("no raw process output reaches the file").**
  A dobby spawned by a pre-change binary has fds 1 and 2 open on `dobby.log` (`start.go:258` doc: "appended to the file at geom.LogPath"), and an exec keeps them.
  After the first re-exec onto the shipping binary, its envelope, Warn lines and any panic still reach `dobby.log`. After a rotation they reach the inode renamed to `dobby.log.1`.
  This lasts until the dobby is respawned, which a dobby living by re-exec never is; `dobby.out`'s "rotated once at each spawn" likewise never fires for it.
  The record is silent on this transition.
- **F4, LOW, design, Decision 8's claim and bound.**
  A state record from before the change already lists other machines' landings in `CountedLandings` and `LandedSinceTidy` (`tidy.go:127`). Only `settleTidy` clears them (`tidy.go:203`).
  So the first tidy after deploy on Neve still runs over the 13 foreign landings the entry's Part 3 reports.
  The record says no start resets the list but never says this residue is tidied once more.
- **F5, LOW, consistency, Decision 2's bound.**
  In the interleave where the re-exec'd process reads the record before the second dobby's `recordStart`, it "exits through Decision 3's refused-re-exec path".
  That path posts a notice naming `lym beam start` while a dobby is in fact running, a false alarm the bound presents as harmless.
- **F6, NIT, consistency, Decision 10's bound.**
  Either spawned check can win the lock. The first `lym adopt`'s envelope may be the one naming a pid that never ran, not only the second's.
- **F7, NIT, consistency, Decision 6's rationale ("within ten minutes").**
  The heartbeat is written by the first tick ten or more minutes after the last one, and ticks are up to `idleCeiling`, one minute, apart (`loop.go:23`). The gap can reach about eleven minutes.

**Tool calls**

1. `internal/logger` stderr policy and the `LandedSinceTidy` writers: to check D5's stderr bound and D8's tidy-count bound, which the manifest lacked.
2. `tidy.go` `countLandings` and the logger's stderr lines: the body of the count D8 relies on.
3. A grep for every dobby lock path user: to look for a long-holding prober for D1 (no conclusive result).
4. `state.go` lock path and `start.go` `Live`: to confirm the probe's try-and-release and that the state lock is a separate file.
