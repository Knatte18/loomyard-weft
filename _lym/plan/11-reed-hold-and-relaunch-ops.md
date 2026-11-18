# Card 11 — Reed ops: end a strand's process while holding it, relaunch it under the same GUID

**Create:**
- `plan:internal/reedengine#RelaunchSpec` -> `type RelaunchSpec struct`
- `plan:internal/reedengine#Engine.EndStrandProcess` -> `func (e *Engine) EndStrandProcess(guid string) error`
- `plan:internal/reedengine#Engine.ReleaseStrandHold` -> `func (e *Engine) ReleaseStrandHold(guid string) error`
- `plan:internal/reedengine#Engine.RelaunchStrand` -> `func (e *Engine) RelaunchStrand(guid string, spec RelaunchSpec) error`
- `plan:internal/reedengine#TestEndAndRelaunchStrand_KeepTheGUID` -> `func TestEndAndRelaunchStrand_KeepTheGUID(t *testing.T)`
- `plan:internal/shuttleengine#fakeReed.EndStrandProcess` -> `func (f *fakeReed) EndStrandProcess(guid string) error`
- `plan:internal/shuttleengine#fakeReed.ReleaseStrandHold` -> `func (f *fakeReed) ReleaseStrandHold(guid string) error`
- `plan:internal/shuttleengine#fakeReed.RelaunchStrand` -> `func (f *fakeReed) RelaunchStrand(guid string, spec reedengine.RelaunchSpec) error`
- `plan:internal/testkit/shuttlefake#Reed.EndStrandProcess` -> `func (r *Reed) EndStrandProcess(guid string) error`
- `plan:internal/testkit/shuttlefake#Reed.ReleaseStrandHold` -> `func (r *Reed) ReleaseStrandHold(guid string) error`
- `plan:internal/testkit/shuttlefake#Reed.RelaunchStrand` -> `func (r *Reed) RelaunchStrand(guid string, spec reedengine.RelaunchSpec) error`

**Extends:**
- `internal/reedengine#Strand`
- `internal/reedengine#StrandStatus`
- `internal/shuttleengine#ReedOps`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine/doc.go`

**Edit:**
- `internal/reedengine/doc.go`
- `internal/reedengine#planResumeLaunches`
- `internal/reedengine#TestPlanResumeLaunches_ThreeLifecycleStates`
- `internal/reedengine#Engine.Status`

**Uses:**
- `internal/reedengine#Engine.launchStrandLocked`
- `internal/reedengine#Engine.RemoveStrand`
- `internal/reedengine#Engine.ReplaceStrand`

**Intent:**
D7's two reed ops; `ReplaceStrand` mints a new GUID, so neither exists today.
`Strand` gains `Held bool` (`held,omitempty`), persisted in `reed.json`, and `StrandStatus` reports it.

- `EndStrandProcess(guid)`, under the op lock, marks the strand held and saves the state, then kills its panes and reaps their process trees as `RemoveStrand` does, and clears its pane id, keeping the record, its GUID and its commands.
  A held strand is never relaunched by `planResumeLaunches` (`Resume` reports it under `Held`), and reconcile, which only kills dead or untracked panes and clears bindings, leaves its record.
  A failure after the mark returns the error with the strand still held; the caller releases it.
- `ReleaseStrandHold(guid)` clears the mark, for D7 step 1's failure path, leaving the pane as it is.
- `RelaunchStrand(guid, spec)`, under the op lock and on a held strand, replaces the stored `Cmd`, `ResumeCmd` and `SessionID` with `RelaunchSpec`'s, launches the pane through `launchStrandLocked` with the new `Cmd`, clears the mark, and saves; the GUID never changes.
  A strand that is not held, or not tracked, is refused, so a stop that removed the strand during the relaunch wins.
- `RemoveStrand` removes a held strand like any other.

Reed resume and reconcile skip a held strand for as long as it is held, so a `Wait` process that dies between D7's step 1 and step 4 leaves the strand down.
The hold ends at the stop verb's removal, `ReleaseStrandHold`, `RelaunchStrand`, or card 15's settle at the next attach or `Wait` past the relaunch bound; nothing else clears it.
`reedengine/doc.go`'s sentences that Resume relaunches every non-live strand with no list or a missing path, and drops every non-live strand whose list is complete, gain the held strand's exception: Resume neither relaunches nor drops it.

`ReedOps` gains the three methods; `fakeReed` records each call in its `CallLog` with a scripted error and runs an optional `OnEndStrandProcess func()` field after recording that call, so a test can interleave a stop, and `shuttlefake.Reed` implements them over its strand table, so `skillReed`, which embeds `fakeReed`, and every `ReedOps` user still compile.

`TestPlanResumeLaunches_ThreeLifecycleStates` (tier0) gains a row: a held, not-live strand is neither launched nor dropped, and is reported held.
`TestEndAndRelaunchStrand_KeepTheGUID` is tier2 on a real tmux server: ending a strand's process leaves the record with its GUID, held and not live; a reed resume does not relaunch it; `RelaunchStrand` with new commands brings up a live pane under the same GUID whose record names the new commands and session id; a `RemoveStrand` between the two makes `RelaunchStrand` refuse and leaves no strand; `ReleaseStrandHold` clears the mark and a later reed resume relaunches the strand; `RelaunchStrand` on a tracked strand that is not held is refused.

**ImpactSummary:** Three reed ops and one persisted flag; `ReedOps` grows by three methods whose implementers are reed's engine and the two test fakes.

**Commit:** 11: reed: end a strand's process while holding it, and relaunch it under the same GUID
