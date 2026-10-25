# Card 15 — innerrun-respawn-cap

**Edit:**
- `plan:internal/battenshed#innerRunMemory`
- `internal/battenshed#innerRunProducer.Call`
- `internal/battenshed#innerRunProducer.noticeCondition`
- `internal/battenshed#conditionText`
- `internal/battenshed#TestNotice_DeadDriverWhileRunning`

**Create:**
- `plan:internal/battenshed#respawnCap` -> `const respawnCap = 3`
- `plan:internal/battenshed#TestInnerRun_RespawnCapStopsWithoutProgress` -> `func TestInnerRun_RespawnCapStopsWithoutProgress(t *testing.T)`

**Intent:**
Cap the respawn of a running child whose driver keeps dying (D4, D9's driver-dead).
`innerRunMemory` gains the respawn count of the running child and the child status file's modification time at the last respawn.
Before `Call` respawns an unconfirmed running child with no driver at work, it compares the status file's modification time with the recorded one: a later write resets the count to zero, and a live driver strand or a held lock alone does not, so a driver that dies minutes after each respawn still reaches the cap.
The recorded time is taken from a stat of the status file after `deps.Spawn` returns, so the bootstrap's own write during the respawn is not read as progress at the next pass.
At `respawnCap` respawns with no status write between them `Call` spawns nothing and goes on to the check, where `noticeCondition` reports the driver-dead condition with the text `driver strand is dead; respawned 3 times without progress; run "lyx loom start" in the task worktree to bring a driver back`, once per episode as every notice is; the count resets, and respawns start again, on a status write or a change of the child's state.
The driver-dead notice also goes when a revive of a halted child's driver strand failed in the episode (the revived marker reads `failed`), with the text `driver strand could not be revived`: for a halted or awaiting child `noticeCondition` reports it once the episode's state-changed key is already kept, so the halt's own notice goes first and each goes once.
`TestInnerRun_RespawnCapStopsWithoutProgress` (tier0, parallel), one producer per pass over one scratch directory, each pass preceded by rewriting the spawn-confirmed marker to a foreign pid so the pass reads as a new process: three passes each find the child running with no driver and respawn it, the fake spawn rewriting the status file as the bootstrap does, the fourth spawns nothing and sends one driver-dead notice, a fifth sends none; a run in which `DriverStrand` reads live between two respawns and dead again at the next pass still reaches the cap; after the fake child's status file is rewritten between passes the next pass respawns again.
`TestNotice_DeadDriverWhileRunning` gains a row for a failed revive's notice text.

**ImpactSummary:** The spawn path of one producer gains a counted guard and the notice step one condition text; no seam or caller outside battenshed changes.
