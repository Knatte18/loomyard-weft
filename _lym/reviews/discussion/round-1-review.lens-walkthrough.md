```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: run each Decision's mechanism end to end on one concrete case and check the end state against the Decision's purpose.

**Decisions walked**

- **D1**, a task with recipe `raddle` started by batten: it runs Worktree-Create, Record-Write, Raddle-Write, Raddle-File and Land. Purpose met.
- **D2**, a Sketch note promoted with `--recipe raddle`: it becomes a released task that keeps its label, flag and record. `IsSketch` already answers false for any task (`sketch.go:35-36`). Batten's start guard finds the stored record. Purpose met.
- **D3**, a filed darn part flagged discussed: Record-Write copies its stored record. This holds only if the part record reached the board store, which fails at D9 step 2 below.
- **D4**, the agent reads `lym report cost`: the JSON form exists by D13. Purpose met, but its cost input is biased (see D13).
- **D5 and D6**, a `tasks` proposal with two parallel darn parts on disjoint packages: it passes the checks. Purpose met.
- **D7 key off**, operator approval in the first session: met. After a D8 return, the second interactive session gets no waiting notice. **Not met.**
- **D7 key on**, auto approval: the proposal file ends the session and the gate passes. Raddle-File then needs a cold proposal that nothing stored. **Not met** (D9 step 1, D14).
- **D8**, a weighed entry whose `revised` stamp changed, caught at filing: Stuck back to Raddle-Write in a new session. Existing producers archive stale outputs before a fresh spawn (`shedadapters/archive.go`), so the new session starts clean. Met, apart from the notice gap above.
- **D9 step 2**, part records for a `tasks` filing: the records must be stored before step 3 creates the entries. The board's one store path refuses an absent entry. **Input mismatch.**
- **D9 step 3**, marking the Sketch done: the run lock ignores Status (`runlock.go:185-217`), and `SetRunStatus` skips a done entry (`board.go:203`). Met.
- **D9 step 4**, a `beam` shape: `lym beam create` must act as the parent worktree (`beamcli/create.go:77-101`), which D9 states. Met.
- **D10**, Land prunes the done Sketch: `MakeSketch` also skips a done entry (`sketch.go:77`), and the raddle store keeps the copy. Met.
- **D11**, the pawl BeamUp of a raddle-filed beam: it mails the beam's orch once, using the sent-marker. Met.
- **D12**, a session ending stuck: it writes one usage file per session id. Met.
- **D13 and D14**, predicting a discussed tapestry part from fits over undiscussed tapestry runs: the prediction misses the comparison's purpose. **Not met.**
- **D15**: a doc row only. Met.

**Findings**

1. **MEDIUM**, design, D9 step 2 (with D3): the part records cannot be stored as written.
   - Step 2 stores each `decision-<slug>.md` before step 3 files the entries.
   - `boardengine.StoreDecisionRecord` is the only path that writes the record file and sets the `discussed` flag, and it refuses an absent entry. It also refuses a done entry.
   - The step therefore has no slug to store on. A part record left only in the run's tree never becomes the stored record that D3's Record-Write reads for darn and pawl.
   - The record should state that the files and their `discussed` flags join step 3's one board write, or that they are stored after it.
   - Evidence: `internal/boardengine/decision.go:38-61` (lines 46-48 refuse an absent entry).

2. **MEDIUM**, design, D7 with D9 step 1 and D14: auto mode never stores a cold proposal or an approval.
   - Only `lym loom raddle-propose` stores the cold proposal (D7), and it runs only with the key off.
   - In auto mode, step 1's input "the cold proposal" does not exist, and nothing defines which file is "approved".
   - After a D8 return, the agent rewrites `proposal.yaml`. D14's cold-against-approved comparison then reads a missing or arbitrary file.
   - The record should state what auto mode stores as cold, for example the first gate pass.
   - Evidence: decision-record.md D7 lines 84 and 91, D9 line 110, D14 line 153.

3. **MEDIUM**, design, D7 with D8: after a return to Raddle-Write in interactive mode, the operator is never told.
   - `raddle-propose` posts the waiting notice on its first pass per run only, and the D7 criterion pins that.
   - A D8 failure at Raddle-File comes after the operator has approved and moved on. The new interactive session waits for a fresh approval with no notice to the parent orch, so the run stalls silently.
   - Fix: a return re-posts the waiting notice. Alternatively, the notice goes out on the first passing propose of each session rather than each run.
   - Evidence: decision-record.md D7 line 84, D8 line 102, criterion line 196.

4. **MEDIUM**, design, D13 with D4 and D14: the cost fit is keyed by recipe and profile only, so filed tapestry parts are mispredicted.
   - The history of tapestry runs is mostly undiscussed runs, whose fixed cost includes the Discussion (Motif) block.
   - Raddle files every part discussed, and a discussed tapestry starts at Plan (Goal, D3, Notes on `seedDiscussedRecord`). Its real fixed cost is lower than the fitted one.
   - The agent therefore overstates every tapestry part's fixed cost. That biases it toward the "fewer, larger parts" rule D4 states, and D14 then reports a systematic error that is not the agent's.
   - The same holds for darn. The record should key the fit by whether the run started discussed, or state that the bias is accepted.
   - Evidence: decision-record.md D13 line 147, D3 lines 40-41, Goal line 3, Notes line 216.

**Tool calls**

- `internal/boardengine/decision.go` lines 1-80: whether the store path refuses an entry that is absent when step 2 runs. My context held only the `decision.go:41` grep line.
- A grep of `internal/loomshed`, `internal/shedadapters`, `internal/shedengine` and `internal/seatengine` for stale-output handling: whether a returned Raddle-Write session meets the previous session's output files. Not in the manifest.
- `internal/beamcli/create.go` `create`, `refuseCreateHere` and `createSteps`: whether D9 step 4's beam create from the raddle run's Go row passes the acting-worktree guard. The manifest named the file but held none of its code.
