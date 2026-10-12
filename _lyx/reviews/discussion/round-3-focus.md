---
round: 3
focus:
  - "In decision-record.md D5 and D11 and the Acceptance criteria, settle who removes a dead-but-present own-window strand outside a pair teardown. In the code, shuttleengine `removeStrandIfLive` (stop.go), `StopStepRuns` (stepstop.go) and `removeSupersededStrands` (attach.go) skip a strand whose reed `Live` is false, and `Wait` removes a strand only on `OutcomeDone` without `KeepPane`; check that the record decides a remover for the prime, for a timed-out or died run with no later stop, and for a seat table whose advisor died, and that each has an acceptance line."
  - "In D3, settle whether `remove` still kills the strand's pane and ends its window when `window_split` is invalid or `window_share_max` is negative. Shuttle's `Wait` cleanup logs a failed `RemoveStrand` as non-fatal, a seat stop fails with `ErrSeatNotStopped`, and D12's chained self-removal is a `remove`; check that the record and the acceptance line for invalid config agree on which ops fail."
  - "In D2, settle whether a strand bound to a dead pane counts toward a group's live count, and when the group's recorded choice and `WindowSize` are dropped, given that D11 names a group by table prefix and round so a re-step reuses the name."
  - "In D12's Bound and the open verb's acceptance line, settle what happens when the chained detached `lyx reed remove` fails: the pane goes dead under remain-on-exit and stays bound under D5, and `planResumeLaunches` in reedengine lifecycle.go relaunches non-alive strands whose done-when paths are not all present, which an opened strand never has."
---

Each entry names a place and a question; none prescribes a fix.
