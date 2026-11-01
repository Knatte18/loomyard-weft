```yaml
lens: board-scope
outcome: reported
```

Task: check the machine-switch decision record against the board (lens: board-scope).

**Findings**

1. `MEDIUM` · class `decision` · D6 (and Scope "Out" bullet 4)
   - **Summary:** D6 lets the adopted beam's dobby start a released task with an empty status right after adopt brings the beam up. The board entry says the verb "starts no run by itself … so nothing runs that the operator does not expect."
   - **Evidence:** The entry body (Design, 2026-11-01) says "It starts no run by itself: every adopted run stays held until the operator lets it go on, so nothing runs that the operator does not expect." D6 starts such a task through the beam steps that adopt itself runs, before any `lym beam go`. Adopt's own beam step brings the dobby up, so a run can start as a direct result of the adopt. This is the surprise the entry rules out, even though no run moved for that task.
   - **Question for the parent:** Should a released task with no run, under an adopted beam, also wait for `lym beam go`? Or does "every adopted run" cover only runs that have a record?

**Goal points of the entry**
- The unit is the beam (or root), and different beams can be owned on different machines: met (D1, adopt subjects).
- The verb does every step: sync Bolt, sync the warp and weft of the named beams, adopt, settle conflicts, start dobby and orch as a pair: met (D1, D2, the beam steps through `orchcli.BeamUp`).
- The old machine releases nothing: met (Scope "Out").
- It starts no run by itself and every adopted run stays held: met for runs with a record (D3). It is in question for released tasks with no run (finding 1).
- The hold is released per beam by a verb, and the reed button calls the same verb later: met (D4). The button is correctly out of scope pending `reed-status-buttons`.
- One line per run (landed, running here, held, owned elsewhere) with its row: met (D5).

**Entries, peers and parent commits checked**
- `dobby-orch-pair-start` (dependency, done): no conflict. D4 starts no dobby, and the Constraints cite `PATTERN-dobby-orch-pair`.
- `reed-status-buttons`: no conflict. The button is deferred to that note, as the entry says.
- `run-pins-its-version`: no conflict, out of scope.
- `dormant-orch`: no conflict. The orch is still started with its dobby.
- `landing-watch-limits`: no conflict.
- `interactive-idle-park`: no conflict.
- Peer `kde-window-placement`: no conflict. It only overlaps on the reedcli and config files.
- Peer `gate-fewer-binaries-trial`: no conflict.
- Peer `plan-split-producers`: no conflict.
- Parent commits: `lym loom peers` lists none.

**Decision notes**
- No `Decision for machine-switch:` note appears in `lym board list`, so there was none to read.
- The record has no `Added after Discussion` entries.

**Tool calls**
- One call read `lym board get` for `dormant-orch`, `interactive-idle-park` and `landing-watch-limits`. My context held only their one-line briefs from `lym board list`, not their bodies, and I needed the bodies to judge whether they conflict with this design.
