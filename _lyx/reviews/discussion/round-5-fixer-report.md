# Round 5 fixer report

Every finding in `round-5-review.md` is fixed.
`lyx loom validate-discussion` exits 0 on the result.

## Changed

All edits are in `_lyx/discussion/decision-record.md` unless named.

- F1 (watched marker and the driver's parent): D6 now writes the marker only when the run's own parent is the prime's orch as well, names the prime batten over a pair created elsewhere (no marker, today's rule, a revive renders unwatched), A7 is narrowed to a pair created from the prime, and the marker acceptance row gains the adopted-pair case.
  Support log: Q31 and a rejected alternative.
- F2 (strand-window seam trust): D7 no longer says the seam trusts a recorded id "after that check accepted the state".
  It states that the generation read fails open, that a recorded id counts as present only in the session's own pane listing, never through a server-wide pane target, and that an absent id is treated as absent; the seam acceptance row gains the other-session id case.
  Support log: Q32 and a rejected alternative.
- F3 (`since` carrier): D4 now says `since` is taken at the first sight of a state, by an in-wait check or a call's baseline stat, held in batten's in-process episode state across calls, pause returns and error returns, and read from the file only at a batten start; the paused-or-failed acceptance row gains a same-state rewrite between two calls.
  Support log: Q28 updated.
- F4 (quiet fallback clock): D5's fallback counts from the later of the newest history entry and `since`.
  Support log: Q33.
- F5 (window verb refusals): the Constraint covers every refusal of the verb instead of "both", D7 names the way forward of a missing `new-window` and of a failed call, the read-back refusal names `lyx batten status <slug>` first, and the window acceptance row names the refusals.
  Support log: Q34.
- F6 (Open risks duplicates): the dead-batten and lost-destination entries are one, and the failed-relay delay is one entry, with the stencil-noise entry stripped of its relay clause.

## Deferred

Nothing deferred.

## Constraint conflicts

None.
