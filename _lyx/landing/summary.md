# Burler round runs as one seat table on the seats engine

Burler's review round now runs as one `seatengine` table instead of two ad hoc shuttle runs.
The fixer is the `chair` (strand `<shortname>:<slug>:burler-chair`) and the reviewer is the required `advisor-1` (`burler-advisor-1`).
`roundHalves`, the parallel `join` and `lyx burler await-review` are gone.
The round's outcome, review files, fixer-report, failure rules and Bouncer contract are unchanged for every review segment, Darn's perch included.

## What changes

- `burlerengine` builds a seat table per round and runs, probes and resumes it through `seatengine`.
  `decideRound` stays the one place the failure rules live and reads the seat results.
  `Geometry` gains `Shortname` and `Slug`, filled by `hubgeom` and `standalonegeom`.
- `seatengine` gains generic seat capabilities:
  - a per-advisor gate and per-seat fork authorization;
  - `Required` and `Accept` on an advisor, with the advisor's terminal shuttle result, wait error and `Accept` error on `SeatResult`;
  - a chair held-turn notice that waits while advisor work is outstanding.
  `Table.Validate` rejects `Required`, `Accept` or a seat gate on the chair, and `Accept` on a non-required advisor.
  A table with no required advisor, such as Discussion's, behaves as before apart from the hold deferral.
- `shuttleengine.Spec` gains a told hold-notice deferral predicate, which the chair deferral rides.
- The two Burler orchestrator stencils become the chair and advisor seat stencils, and the fixer waits with `lyx shuttle await`, capped under the prompt-cache lifetime.
- `lyx burler await-review` and `internal/burlercli/await.go` are removed.
- The `loom.yaml` key `fix_start` and its `after-review` value are retired; a hub file still holding the key reconciles and loads.
- Docs follow in `doc.go` of `burlerengine`, `seatengine`, `shuttleengine` and `burlercli`, in `docs/overview.md` and in `tools/sandbox/SANDBOX-BURLER-SUITE.md`.

## Design decisions

- Burler keeps its own Shed row and `BurlerProducer`; only the agent wiring moves onto the seats engine.
- Review-before-fix stays a Go gate: a required advisor's `Accept` runs the cluster audit and strict parse, then writes the ready marker.
  Any way the required advisor ends without `Accept` succeeding stops every started seat and ends the round, so the chair never waits on a marker that cannot come.
- Start order is the seats engine's, chair first.
  A stale ready marker is removed before the chair starts.
  A chair that fails to start starts no reviewer; a reviewer that fails to start stops the chair.
  `fix_start` is retired because `after-review` would need the engine to defer the chair behind an advisor.
- Lenses stay forks of the reviewer seat: `ForkSubagents` is set on the reviewer only when the profile names a cluster fan.
- The Burler seat stencils include neither shared seat directive, because those lines contradict Burler's narrower channel.
  The chair may message the reviewer only after the ready marker exists, to ask what an accepted finding means.
  The reviewer sends nothing unprompted and writes nothing after its review.
- A required advisor keeps its hold notice (`QuietHold` false), since the reviewer is the round's critical path.
- The chair's held-turn notice is deferred only while a live advisor is mid-turn or its up line is pending.
  It is delayed by a settle interval, never dropped, and capped by a maximum deferral; both are constants in `shuttleengine`.
- Resume attaches only when the chair and reviewer are both live; any other combination goes back to `BurlerProducer` for a fresh round.
- The strand names change to `burler-chair` and `burler-advisor-1`.

## To check by hand

- The `llm`-tier smoke tests were compiled but not run.
  Run `internal/seatengine/smoke_seats_test.go` and the `internal/burlerengine` round and cluster smokes.
  In particular, confirm that an advisor turn started by the chair's SendMessage reads as mid-turn by shuttle's session-state fold; if the provider's turn-start hook does not fire, the chair's notice fires after the settle interval.
- Hold deferral bound: check that the maximum deferral in `internal/shuttleengine/hold.go` actually bounds a chair stall hidden behind a wedged advisor (finding `chair-notice-deferral-bound-vacuous`).
- Early end: check in `internal/seatengine/engine.go` that a required advisor ending early is itself stopped along with every other started seat (finding `required-advisor-not-stopped-on-early-end`).
- Probe: check what `ProbeRound` in `internal/seatengine/resume.go` and `internal/burlerengine/resume.go` stops when only the reviewer is live (finding `probe-round-reviewer-only-stop-undecided`).
- `seatengine`'s `join` now also stops a chair that timed out, so a timed-out fixer does not hold its strand name against the next attempt.
- Stencil-only rules are not enforced by Go: the chair sending nothing before the marker, the reviewer sending nothing unprompted, the reviewer writing no file but its review, and the fixer acting on nothing beyond the accepted review.
  The kept reviewer keeps its edit tools through the fix, so such a write would land in the fixer's commits unreviewed.
  Go catches only an edit to the review file.
- Deploy when no Burler round is in flight: a fixer mid-round across the deploy calls the removed `lyx burler await-review` and fails, and the row's retry or a re-step recovers.
- A round now holds two panes until it ends, since the reviewer's pane is kept.
  A reviewer that never comes up costs one started-and-stopped fixer.
