---
round: 2
focus:
  - "internal/dobbyengine/runnotice_test.go and runnotice.go (saveNoticeMemory, episode since, api-error revive streak): does any test fail when the notice memory is not saved or not read back across a dobby re-exec over the same state record, for both the episode marker and the revive streak?"
  - "internal/dobbyengine/runnotice.go run selection (cancel marker, non-running batten state, absent child status, ReadChild error, nil ReadChild): does each skip have a test row that fails when that skip is removed, and does a ReadChild error leave the tick succeeding?"
  - "internal/battenshed/innerrun_memory.go RespawnCapReached and internal/battencli/dobbynotice.go readNoticeChild: can a status write after the last respawn still yield a respawn-cap driver-dead text, and does an unreadable memory record still suppress every condition of the run?"
  - "internal/shedrecipe/entries_batten.go and internal/beamcli/dobbydeps.go runShedDuration: what does an explicit zero notice_quiet_min now do compared with before the change, and is that behaviour stated or tested?"
---

Focus covers the gating test gaps and the two behaviour questions the first review left; each entry is a question to settle against the diff, not a verdict.
