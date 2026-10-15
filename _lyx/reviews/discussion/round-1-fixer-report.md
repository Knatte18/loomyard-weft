# Fixer report — discussion round 1

Every change is in `_lyx/discussion/decision-record.md`.

## Changes

- **F1 (BLOCKING), fixed differently.**
  D3's second bullet now gives the repeated-conflict halt a reason that names the way forward: edit the named pending cards by hand so they match the tree, then `lyx loom resume`, which re-steps the Webster row, whose auto-rebaseline accepts the edited cards.
  D5 states that the loom row for this halt carries the same way forward, and the adapter's Acceptance criteria clause pins that the halt's reason names it.
  I left out the proposed manual `lyx webster rebaseline --card NN` step.
  The refusal spec's "Webster row: auto-rebaseline refused" row (`contracts/specs/refusal-spec.md`) says a way forward under the Webster row is "never a manual `lyx webster rebaseline` or `lyx webster run`", and D3 already says the Webster row's loom entry sets `AutoRebaseline`.
  `lyx loom resume` is the verb the loom section names for re-running a halted row, as the "Darn-Write verify gate exhausted" row shows.
- **F2 (MEDIUM), fixed as proposed.**
  In D2, every begin-batch call removes a present record first, under the mutate lock and before its re-resolution, and writes it again only on a `plan_conflict` refusal.
  The Rationale says why.
  D3's Bound paragraph names the remaining exposure: a master that goes stuck for another reason after the refusal is still rerouted, and D4's stop-at-once rule bounds that.
  A new Acceptance criteria clause says a begin-batch refusal other than `plan_conflict` leaves no conflict record.
- **F3 (MEDIUM), fixed as proposed.**
  D3's Bound paragraph no longer says "the Plan-Review ring keeps its own bounce budget".
  It now says the reroute resets Shed's bounce count while the Bouncer's hard cap reads the round number, which continues as N+1, and gives the soft-cap consequence: an auto-accept back to Webster, or a hard-cap escalation on a design-wrong CONTINUE.
  The Open risks bullet on the inherited ledger and cluster exclusions now names the same consequence.
- **F4 (MEDIUM), fixed with the first option.**
  D3 compares the set of `Finding.Error` strings as D2 writes them, ignoring order and leaving out the batch number and cause.
  The Bound paragraph and the Rationale now say that one plan review is the bound only when the edit leaves the findings' card ids and numbers unchanged, and that shed's reroute cap bounds a renumbered conflict.
  The Rationale also says why this option won: the record stays plain text, with no second key to keep in step with the checks' detail wording.
  The adapter's Acceptance criteria clause pins that a reordered but otherwise equal set counts as equal.
- **F5 (LOW), fixed as proposed.**
  D1 now says the begun set is `DispatchScope`'s, computed before this begin-batch records its batch, so a re-begun first batch is `tree_moved` whether or not its fork landed work.

## Disputed

None.

## Deferred

None.
