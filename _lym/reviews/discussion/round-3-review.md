---
verdict: BLOCKING
findings:
  - id: F26
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:220
    carries: F22
    summary: "D7 step 4 says the relaunched pane passes the start path's startup gates, but the run's events file is reused, so awaitStartup marks the pane started on its first tick from the earlier session's lines and plays no gate"
    fix: "In D7 step 4, replace the sentence The pane passes the start path's startup gates (`awaitStartup`) with a sentence stating that the relaunch's startup wait decides on the pane's capture alone: the run's events file (`RunState.EventsPath`) is kept and already holds the earlier session's session-start and turn-start lines, so `awaitStartup`'s `checkLivenessTick(..., eventsProof=true)` short-circuit (`turnStartedAfterSessionStart`, `internal/shuttleengine/wait.go`) would mark the pane started before it booted and a trust or login dialog on the new account would never be dismissed. State that the relaunch runs the startup wait with the events proof off, as Wait's own probe of a not-yet-started run does, so only a ready pane capture marks it started; a stale session-start line satisfying the line half is already accepted by `checkLivenessTick`. Add an acceptance criterion under D7: a relaunched pane that shows a trust dialog has it dismissed before the relaunch completes, and a pane that never reaches ready ends the run as a strand that died."
  - id: F27
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:231
    summary: "D7's bound ends a relaunching record only when its strand is not live, so a Wait process that dies after reed has relaunched the pane but before step 4 writes the record leaves a live pane under a record that still reads relaunching with the old session id, prompt offset and launched-account facts, and no rule clears or replays it"
    fix: "In D7, make the record write precede reed's relaunch: since the GUID never changes, step 4 first writes the launch's session id, `PromptLine`, the prompt offset (the events file size at that moment) and the copy fact into the run record in one hold of the record's lock, then has reed relaunch the pane, then clears the mark to running. A failure of the reed relaunch then falls under the existing rule that a failure in steps 2 to 4 ends the run as a strand that died. Extend the bound sentence so it also covers a live strand: an attach or Wait that reads a relaunching record whose strand is live, once twice the startup timeout has passed since the mark, clears the mark to running and attaches, the record already naming the new launch. Add an acceptance criterion under D7: an attach after the Wait process is killed between reed's relaunch and the mark's clear attaches the live run, reads the new session id and prompt offset from the record, and does not read the limited turn end again."
  - id: F28
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:224
    summary: "D7 and the PATTERN bullet make only attach and Wait read a relaunching record as live and only the two stop forms safe, but the shed stop chain's `StopStepRuns` skips every record whose outcome is not running and every strand that is not removable, so a step stop during the window leaves the run, which the relaunch then brings up, and other readers of a running record misread it"
    fix: "In D7's paragraph on record writes and stops, add that every reader of a run record that tests for a running run reads a relaunching record as running through one predicate on the record, and that `Runner.StopStepRuns` (`internal/shuttleengine/stepstop.go`, which filters on `rs.Outcome != runOutcomeRunning` and `strandRemovable` before it calls `stopRecorded`) stops a relaunching run whatever its pane state, as the stop verb already does for a held strand. State the rule, not a list of readers; the sites to change are the ones that compare an outcome to `runOutcomeRunning`: `stepstop.go`, `dispositionCandidate` in `attach.go` (a live-or-held relaunching record must not fall to respawn-eligible), the readers in `activity.go` and `sessionread.go`, and `unfinished` in `carryover.go`. Add an acceptance criterion under D7: a `StopStepRuns` during a relaunch leaves the run stopped and no strand up, and a sibling Start's carry-over read does not treat a relaunching run as unfinished."
prior:
  - id: F20
    status: resolved
    answer: "D7 now bounds the mark at twice the startup timeout since the mark, ends the run as a strand that died past it, keeps the run directory through the orphan sweep, releases reed's hold and clears the mark when step 1 fails, and re-reads the record in one hold of its lock for every relaunch write and both stop forms; the fixer's twice-the-timeout bound is sound against the sweep's own `minAge`. The live-strand crash window it leaves is F27, and the stop chain's filter is F28."
  - id: F21
    status: resolved
    answer: "D7 keeps the strand's GUID and has reed relaunch the pane under it, with an acceptance criterion for `StopStrand` and attach given the pre-relaunch GUID; `FindRun` matches on the unchanged `StrandGUID`."
  - id: F22
    status: open
    answer: "The first turn is settled: a continue line for a run with skills and without, no skill-load turn, an empty `PromptLine`, the prompt offset set at the relaunch. The startup-gate half stays open because the stated mechanism, `awaitStartup`, passes vacuously on the reused events file; carried by F26."
  - id: F23
    status: resolved
    answer: "D6 now has the resume check report whether it copied, the start path carry that fact into the launch and write it into the new run directory after `createRunDir`, D7's relaunch write it, and a retrying caller carry the first check's report; the acceptance criteria cover the shuttle and the orch `StartSession` paths."
  - id: F24
    status: resolved
    answer: "D7's orch paragraph now names the `StartRelaunch` body in `internal/orchcli/start.go`: withdraw, remove the old strand, `launchOrch`, register, and save the reset onto the new GUID with no second watcher. That matches the code, and the watcher already rebinds when `st.Strand` changes (`w.strand != st.Strand` in `internal/orchengine/watcher.go`)."
  - id: F25
    status: resolved
    answer: "D5's bound now says forged high figures on every other account make a limited worktree wait its limit out, forged low figures on an exhausted account make switches take it until a limit hit writes `blocked_until`, and no forged input writes or clears a `blocked_until`; D4's beams-pin skip reads only `blocked_until`, so the bound holds."
---

### [BLOCKING] D7's startup-gate claim is vacuous on the reused events file

**Location:** `_lym/discussion/decision-record.md:220`
**Issue:** Step 4 says the relaunched pane passes the start path's startup gates through `awaitStartup`.
`awaitStartup` calls `checkLivenessTick` with `eventsProof` true, which marks a `RecordsSessionStarts` run started once the events file holds a session start followed by a turn start.
Its own doc comment says that holds only because the launch's events file is created fresh.
The relaunch keeps the run's events file, which already holds both lines from the limited session, so the first tick marks the pane started before it has booted.
No trust or login dialog the second account's directory shows is dismissed, and the Decision's acceptance line "the relaunched pane passed its startup gates" is satisfied by nothing.
This is the startup-gate half of F22, carried.

### [BLOCKING] A Wait that dies after reed's relaunch leaves a stale record under a live pane

**Location:** `_lym/discussion/decision-record.md:231`
**Issue:** The bound ends a relaunching record whose strand is not live.
Step 4 has reed relaunch the pane first and then writes the session id, `PromptLine` and prompt offset and clears the mark.
A process killed between the two leaves a live strand, so the bound never fires, and a record that reads relaunching with the old prompt offset and session id.
An attach reads it as live, replays the limited turn end from the old offset, and a fresh-launch fallback's new session id appears nowhere in the record.
Because the GUID is unchanged, the record can be written before reed acts, which removes the window instead of bounding it.

### [BLOCKING] The step stop and the other readers of a running record skip a relaunching run

**Location:** `_lym/discussion/decision-record.md:224`
**Issue:** D7 makes both stop forms read and write the record in one lock hold and has the stop verb remove a held strand whatever its pane state.
The shed stop chain reaches a run through `StopStepRuns`, which skips a record whose outcome is not running and a strand that is not removable before it ever calls the stop verb.
A step stop during the window therefore leaves the run alone, and the relaunch then starts the pane the stop meant to end.
`dispositionCandidate`, the readers in `activity.go` and `sessionread.go`, and `unfinished` in `carryover.go` test the same outcome, and the Decision names only attach and Wait as reading a relaunching record as live.
