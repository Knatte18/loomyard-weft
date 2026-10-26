# Card 19 — batten-quiet-at-beam-await

**Edit:**
- `internal/battencli#readAgentActivity`
- `internal/battencli#TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker`
- `internal/battenshed/doc.go`

**Intent:**
D9: batten's InnerRun sends no quiet notice for a child at Beam-Await, as it sends none while a parent lock wait record stands; a restarted batten re-spawns the driver, which re-enters the row.

`readAgentActivity` reports `waitLive` true when the child's own status file (`shedrun.StatusFile(taskLocation, shedrun.SelfRunID)`) reads `running` at `pawlengine.NameBeamAwait`, beside the verify marker, the parent lock wait record and the shuttle wait markers it reads today, so `judgeAgents` finds no quiet condition there; an unreadable child status reads as no such wait, as an absent one does.
The driver-dead notice is untouched and applies at Beam-Await as at any row: a probe that finds the running child's driver dead sends it once per state episode; the respawn cap changes only its text, its episode key the same at and below the cap; and a re-spawned driver that writes its status file on re-entering the row resets the respawn count.

`TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker` gains a child at `running · Beam-Await` with no agent, which reads `waitLive`, and a child at `running · Parent-Align`, which does not.
The driver-dead behavior at Beam-Await needs no new row: `battenshed` reads the child's producer as a plain string, so `TestNotice_DeadDriverWhileRunning` and `TestInnerRun_RespawnCapStopsWithoutProgress` already pin one notice per state episode and the respawn reset for a child at any row, Beam-Await included.
`battenshed/doc.go` names the Beam-Await wait among the waits that silence the quiet notice.

**ImpactSummary:** one more wait source in batten's activity reading, read only for a child at Beam-Await; every other child's notices are unchanged.

**Commit:** 19: No quiet notice for a pawl run waiting at Beam-Await
