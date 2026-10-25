# Fixer report — Discussion-Review round 1

Every non-`decision` finding was checked against the code and held; all are fixed in `_lyx/discussion/decision-record.md`, with twins in `_lyx/discussion/support-log.md`.
Separately, the parent's `Added after Discussion` entry (creel line from `LOOM`) ordered the Open risks line on a landing-time approval dropped; it is dropped.

## Changes

- F1 (fixed, as proposed, then reshaped by the self-check): D8 keeps the escalation ahead of the refusal for routing only, and an `accept` on a `decision` escalation applies the judgment refusal before `acceptRound`, leaving the round's `decision` findings out of its BLOCKING check.
  A refused accept is not passed: below the hard cap it runs as a `continue`; at or past it the accept is retired and the round re-judged, bounded by the bounce budget.
  The brief lists every BLOCKING review finding and the refusal cause; the bounds line is rewritten; an acceptance criterion covers a CONVERGED round with a BLOCKING `design` finding.
  Fixed differently from the proposal's "re-judges instead of passing" alone: a re-judge leaves the review unchanged (`rejudge` archives only `judgeOutputs`), so it would loop on the same refusal; the self-check forks showed this.
- F2 (as proposed): D8's second bullet limits the never-settled rule to a decision file absent or not yet settled; a settled accept is a settled generation again. Criterion added for re-entry after a settled accept.
- F3 (as proposed): D6 gives `goal-scope` `lyx board list`, the task entry and up to four decision notes, newest first; `board-fit` counts note reads inside its four reads and names notes from the list titles. Each list names notes read and not reached. D5's fit sentence corrected; criterion extended.
- F4 (as proposed): D11 adds the `Parent-Align` fallback to the `Merge-In`-named fixer attempt record and its removal; criterion added for a run halted spent or over-cap.
- F6 (as proposed): D8's hard-cap bullet keeps cause `decision` on a hard-cap re-escalation; criterion added that it stays refused to the arbiter.
- F7 (as proposed): D8 gives the fix row its own key, set with the Bouncer's on loom's Discussion rows and pinned by a recipe test; the Constraint line and Auto-mode assumption name both switches.
- F8 (as proposed): D10's writer asks the parent for the note and a `SendMessage` reply, and re-reads the board on any parent message; with no parent it builds nothing and writes the change description as its stop report. Bound line, criterion, support-log Q11 and a plan-writer note follow.
- F9 (fixed, own choice among the proposed outcomes): D5 states the `discussion` fan runs the rewritten `goal-scope` and a hub-tuned lens keeps its own text, unreported by reconcile. Criterion added.
- F10 (fixed differently): the proposal's "carry only when no answering entry exists" needs a semantic judgment the carry-over code cannot make; instead the round's first `decision` escalation snapshots the `Added after Discussion` titles, re-escalations carry the snapshot over, and each carried finding names the entries recorded since. Criterion adjusted.
- F11 (as proposed): D6's `board-fit` bullet narrowed to a contradicting commit or one the writer cannot settle; an already-made decision the writer can cite or drop is an ordinary finding.
- F12 (as proposed, first option): D8's bounds line states D7's bound is reviewer discipline enforced by the rubric only.
- Support log Q8 updated to D8's revised rules.

## Self-check

- fix-consistency, MEDIUM, refused accept re-judge loops and contradicts the accept criterion: fixed (merged with the fix-way-forward, fix-claims and fix-proposal F1 findings; see F1 above).
- fix-consistency, LOW, D10's let-through line stale: fixed.
- fix-consistency, LOW, Auto-mode assumptions name one switch: fixed.
- fix-consistency, LOW, support-log Q11 stale: fixed.
- fix-way-forward, MEDIUM, no exit for a refused accept at or past the hard cap: fixed; the re-judge's bounce unit halts at budget, now stated, and below the cap the accept runs as a continue.
- fix-way-forward, LOW, snapshot reset on re-escalation: fixed; the first escalation's snapshot is carried over.
- fix-claims, LOW, the re-judge's bounce cost is unstated: fixed in D8's bounds line.
- fix-proposal, MEDIUM, F1's re-judge cannot clear the refusal: fixed (merged above).
- fix-proposal, MEDIUM, no-parent path builds an outdated entry: fixed; the writer now builds nothing and stops with the change description.
- fix-proposal, LOW, snapshot lost on re-escalation: fixed (merged with fix-way-forward's).
- fix-proposal, LOW, four-note cap can stop a run on an answered question, and "nearest first" undefined: fixed; notes read newest first, and D8's bounds line names the miss.
- fix-proposal, LOW, "knowingly" is unsupported: fixed; the sentence says unreported.

## Disputed

None.

## Deferred

None.
