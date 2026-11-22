---
verdict: CONTINUE
rationale: "Round 1 carries BLOCKING F2 (archive tag of a discarded task branch pushed to plain refs/tags/) and design-class MEDIUM F1, F4, F5, F6, so the decision rule gives CONTINUE; round 1's fixes were never seen by a fresh reviewer and cannot count as convergence."
---

## Rule applied

The latest review's top-level `verdict: BLOCKING` is no signal; the decision comes from the findings.
F2 is BLOCKING, so the rule gives `CONTINUE` on its own.
F1, F4, F5 and F6 are `design` class at MEDIUM, which is gating by the same rule.
F3 (MEDIUM) and F7 (LOW) are `consistency` and not gating.
No finding is relabelled; every class and severity stands as the review wrote it.

## Convergence conditions

- Gating findings at MEDIUM or worse: five (F1, F2, F4, F5, F6).
- Live gate: green, and the copies run passed, with no strays.
- Reviewer model differs from the previous round's: there is no previous round, so the condition cannot yet hold.
  Round 2 needs a reviewer model other than `opus`.
- The six fix commits postdate the review and are new text no fresh reviewer has seen.
  They count for nothing toward convergence until round 2 re-drives them.

## Focus departures (ratified or rejected)

The review lists nine departures.
Each is a focus entry the reviewer did not drive; none is a deliberate change of the operator's focus, and none steered outside it.

- Sandbox exception, symlinked and `..`-relative paths and a hub moved out of the root: rejected as a departure.
  It is undriven work, and round 2 drives it.
- Joining a mid-seed origin, a legacy `lym-bolt` board and a warp-bound default branch: rejected; undriven, carried forward.
- Namespace-leak enumeration of literal `refs/heads/` and `refs/tags/` call sites on a records-store path: rejected; undriven.
  F2 is one site of that class, which is why the sweep is now due.
- Landed-pair and abandoned-run archive tags: ratified for this round only.
  Landing needs a merge run the charter does not build from fabric verbs alone; round 2 drives what the fabric verbs can reach and names the rest under Coverage.
- Two clones at once with `kill -STOP`, and the second pair of hubs: rejected; the review made no concurrency claim, so nothing is graded, but the drive is owed.
- Remote delete with a move from a second clone between read and delete: rejected; undriven.
- A pair verb killed mid-operation, other than `remove` during the archive tag push: rejected; undriven.
- A long-lived process's namespace cache: rejected; undriven.
- Reconcile, prune and cleanup in each task state: rejected; only two states were driven.
- Sabotage proofs: ratified as the reviewer's reading of the rule that a review half changes no source file.
  The rubric still requires a green drive to be shown to enter its mechanism, so round 2 asks for the proof by instrumenting or by a control hub, never by editing source in the review half.

## Coverage

Not driven in round 1: the departures above.
The method also left three things unshown: no scenario was proven to reach its mechanism under sabotage,
no concurrency claim had a sequential control, and the fixes of F2 to F6 were never re-driven against a sandbox hub.
