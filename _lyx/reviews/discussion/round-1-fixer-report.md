# Round 1 fixer report

Every finding was fixed in `_lyx/discussion/decision-record.md`; `support-log.md` is unchanged.

## Changes

- F1: D5 now tells the engine the marker path with the round's other paths as a pure function of the review path, under `.lyx` at the mirrored subpath.
  The caller removes any existing marker before an attempt spawns either half, and D5's bound adds that no marker survives into the next attempt.
  The `PATTERN-told-geometry` and `PATTERN-lyxdirs-single-declarer` constraints follow, and a criterion states that an earlier attempt's marker never releases a later fixer.
- F2: D6 gains a resume paragraph and a new title.
  Both halves live attach to both; one live, or one gone and the other done, stops the live half, archives outputs and the marker, and respawns both as attempt 1.
  The resume criterion follows.
- F3: D1 states that Go waits on both runs concurrently and the first terminal event decides.
  D6 states that an exhausted reviewer gate returns outcome done with the failing gate, no verdict and the fixer stopped.
  D7 states that `Gate` carries the failing half's outcome and puts the reviewer's gate into the parse ordering.
  The failure criterion follows.
- F4: D8 states that no Go code reads the fixer-report's sections, and the `Disputed` criterion is reworded to what is testable.
- F5: D9 says entries are parsed at config load and resolved by `ResolveReview` at run start.
- F6: one criterion added for the per-half timeout, the fixer-only friction directive, and the skills and PATTERN directive on both halves.
- F7: D9 states that each half's deadline counts from its own spawn, so the fixer's waiting counts against it.
- F8: the two Auto-mode assumptions restating D8's rationale and the defaults' provenance are removed; the support log's Q9 keeps the provenance.

## Deferred

None.
