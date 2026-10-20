# Goto, resume and stops keep what a run already settled

`run-reentry` · Task · bug, enhancement, shed, loom, burler, shuttle

Depends on: [`watchdog-reap-test-xdr`](design-watchdog-reap-test-xdr.md), [`lyxbin-timeout-test-xdr`](design-lyxbin-timeout-test-xdr.md), [`lenses-module`](design-lenses-module.md)

## Goal

A goto, a resume or a stop keeps what the run already settled: converged verdicts, spent review budget and the reason a run is paused.
The run `ubuntu-2404-support` (2026-10-19) showed goto and the stop path as the weakest part of shed: one wrong goto cost a fresh review round over an approved plan, a friction session, tackler and arbiter sessions per later stop, and a paused state nobody could explain.
Operator decision (2026-10-19): goto is extremely fragile; once an artifact is approved, no further review runs on it.
Operator decision (2026-10-20): the notes below are fixed together in this one task.

When this lands, the hub orch rule "A review segment that already converged" (`_board/_lyx/orch/rules.md`) is removed through `lyx orch rules`.

## Goto as a controlled re-entry (orch finding, 2026-10-20)

### Seen

`lyx loom goto` and `lyx shed goto` move a run onto a named row and give the whole segment a fresh budget.
They do not know what the segment already settled: a converged verdict is discarded, the spent rounds no longer count, and the run lands in `paused` with no reason recorded.
Every goto-related failure in the notes below follows from that.

### Idea

A goto is a re-entry with defined carry-over, not a raw jump:

- it keeps every verdict the target segment already reached, and a converged review is never re-run by a goto;
- it keeps the segment's spent budget and adds at most the small amendment allowance `amend-review-budget` describes;
- it records itself as the reason of the `paused` state it leaves;
- it resets only what the target row needs to run again, and the goto's help and way-forward texts say what is kept and what is reset.

## Note `converged-bouncer-budget`

### Seen (2026-10-19, run `ubuntu-2404-support`)

The Plan review round converged (`round-1-bouncer-verdict.md`: `CONVERGED`).
Plan-Bouncer then failed on its own commit of the approved plan (a pathspec bug, fixed on main as `7a3396d05`).
Each retry of that commit counted against the Plan-Review segment's bounce budget, so once the fix was deployed the run blocked with "bounce budget exhausted for Plan-Bouncer", whose way forward named `lyx shed goto self --to Plan-Bouncer`.
The orch took that goto; it gave the whole segment a fresh budget, the converged round no longer counted, and the Bouncer started a new review round over an approved plan.

An order to the reviewer to write a review with no findings did not help either: burler refused the round ("requested 3, spawned 0") and the step failed.

### Operator decisions (2026-10-19)

- Goto is extremely fragile.
- Once the plan is approved, no further review runs on it.

### Idea

- A stop after a converged verdict, such as a failed commit or other mechanical step of the Bouncer, does not spend the segment's review budget: retrying it re-runs only the step that failed.
- When the budget does run out at a Bouncer whose round converged, the error names `lyx loom circling accept` and `lyx loom resume`, never a goto, since a goto discards the verdict.


## Note `amend-review-budget`

### Seen (2026-10-19, run `ubuntu-2404-support`)

The plan had passed three Plan review rounds when Webster halted on card 2.
The fix was a small amendment to that one card, made through `lyx loom goto --to Plan-Write`.
The goto reset the Plan segment's budget, so the amended plan faced the full fresh review budget again (2 rounds, hard cap 3) for a change to one card.
The orch cut it to one round by hand in the pair's `.lyx/config/loom.local.yaml` (`plan_max_bounces: 1`, `plan_hard_max_bounces: 1`), on the operator's order.

### Operator view, not a decision (2026-10-19)

A lightly revised plan does not need three new review rounds.

### Operator decision (2026-10-19)

Taking another planner round does not reset the review budget.

### Idea

A segment re-entered for an amendment of an already reviewed artifact keeps its spent budget and gets at most a small allowance on top, one round by default, configurable in `loom.yaml`, never the full fresh budget.
The review of such a round may be scoped to the change against the reviewed artifact.

### Seen again: a PR-Gate reject in darn (2026-10-20, run `gate-fewer-new-binaries`)

The run had passed its review round, Merge-In and Publish when the alignment agent rejected at PR-Gate with one finding: the decision record's Scope bullet did not name the loom plan verify.
The fix was a wording change plus two doc lines, yet the reject re-spawned Darn-Write and the run went through Darn-Bouncer and a whole new Darn-Burler-Review round, then verify and Publish again, tens of minutes on OSL-1046.
Operator view (2026-10-20): a whole new review round after such a small fix is overkill.
The same rule covers this re-entry: a rework after a PR-Gate reject, in darn's Darn-Write and loom's PR-Rework, keeps the review segment's spent budget, and its check may be scoped to the reject's findings and the diff since the reviewed head.


## Note `paused-state-reason`

Imported from [issue #759](https://github.com/Knatte18/loomyard/issues/759).

### What happened

The Go driver found a run in state `paused` with no recorded reason, current producer `Plan-Bouncer`, 32 history entries.
It wrote a `go-driver-paused` friction note pointing at a `drive-reports/drive-envelope-*.json` envelope and nothing else.
This came right after the same run halted `failed` at `Plan-Burler-Review` (a reed "did not track strand" liveness failure) and the history shows a `Plan-Bouncer` stuck entry as the last record.

### Why it is abnormal

A paused run must say who or what paused it.
With no reason, neither the operator nor the driver can tell an intended pause from a lost handoff, and the driver cannot decide whether to resume.

### What to do

- Make every transition into `paused` record a reason (operator request, hibernate, budget, crash-resume, ...), and have the driver refuse or label a paused state that lacks one.
- Check whether the failed-then-paused sequence at the same loom run (`Plan-Burler-Review` failure, then a `Plan-Bouncer` pause) is a path that writes `paused` without a reason.

Provenance: task `ubuntu-2404-support`, notes `go-driver-paused.md`, `loom-halt.md`.


## Note `burler-zero-fork-round`

Imported from [issue #756](https://github.com/Knatte18/loomyard/issues/756).

### What happened

A `lyx loom run` halted `failed` at `Plan-Burler-Review` (anomaly `producer-hard-failure`) with:

`shedadapters: Plan-Burler-Review (burler-review): round 1 attempt 1: review: burler: the cluster round did not produce exactly the requested number of fork reviewers (requested 3, spawned 0)`

The trace shows the review round starting with `clusterFan=review-plan forkCount=3` and `forkSubagents=true`.
About three and a half minutes in, the shuttle logged `session state ... state=idle-stalled cause=interrupt` followed by `shuttle: session state disagrees` (`loop=running`, `stands=loop`).
The session went `busy` again a second later and, eleven seconds after that, reached `idle-done cause=done`.
The shuttle run then finished `outcome=done` and the burler logged `review joined reviewOutcome=done` and `seatengine: step joined chairOutcome=done advisors=0`, yet no fork reviewer had been spawned.

### Why it is abnormal

The chair session reported a clean `done` while the round had spawned none of the 3 requested forks, so the run halted `failed` instead of retrying the round or reporting why the chair stopped short.
The interrupt and the state disagreement immediately before the finish suggest the chair's turn was cut short and its `done` was accepted without the fork count being checked first.
The halt message gives no indication of whether the chair never tried to fork, tried and failed, or was interrupted.

### What to do

- Check the fork count before accepting a chair `done`, and treat a chair that ended with zero forks as a stalled round that can be retried or nudged, not a hard failure.
- Record in the halt note why the chair ended (interrupt, done without forks) so the cause is distinguishable.
- Look at how an `idle-stalled cause=interrupt` followed by a quick `done` is handled in the shuttle's session-state reconciliation.

Provenance: task `ubuntu-2404-support`; notes: `loom-halt.md`.


## Note `friction-not-per-stop`

### Seen (2026-10-19, run `ubuntu-2404-support`)

The run stopped many times in one evening (Webster card halt, Plan-Bouncer commit failure, the review re-run after a goto, the interrupted review).
At each stop the Go driver started a friction agent session (strand `<shortname>:<slug>:friction`), and often a tackler and an arbiter after it.
The friction session held the driver lock while it ran, so `lyx loom resume` was refused ("driver still holds its driver lock and is settling the stop") until it ended; one friction strand the operator stopped by hand stayed registered live and kept the step waiting until the orch removed the strand.
So a single wrong goto cost not only a review round but a friction session, plus tackler and arbiter sessions, per stop that followed.

### Operator view, not a decision (2026-10-19)

Friction does not need to run at every stop.

### Idea

Friction notes are gathered once, by the run's Friction-Reflect at the end, from what the stops left in the run's records; no agent session starts per stop for friction.
A stop never waits on a friction session before the run can be resumed.


