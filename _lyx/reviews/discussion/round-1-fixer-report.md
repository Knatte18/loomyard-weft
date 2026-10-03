# Round 1 fixer report — Discussion

Every finding in `round-1-review.md` is fixed in `_lyx/discussion/decision-record.md`, with the matching support-log updates.
`lyx loom validate-discussion` exits 0 after the edits.

## Fixed

- **F1 (MEDIUM), D1 legacy aliases at harvest.**
  D1 now scopes the aliases to a verdict on disk at Call entry (the clear and the replay) and adds a `Bound:` line: a legacy word harvested from a judge the current Call spawned or attached never settles, and the round is re-judged.
  The verdict-parser acceptance criterion is split to match, plus a new criterion for the harvest case.
- **F2 (MEDIUM), parse-error latest round.**
  D6 now says a latest round with a parse-error facts row gets `CONTINUE` and can never reach `CONVERGED`, and that a parse-error row is no evidence for `CIRCLING`.
  A new acceptance criterion checks that the judge prompt states it.
- **F3 (MEDIUM), departure bounds.**
  D6's departure bullet now says a departure relabels one finding in either direction, and only from what the review file says about that finding.
  A BLOCKING-severity finding is never relabeled downward, and the rationale names every departed finding's ID.
  The lead sentence points at that bullet instead of restating it, and the Open risks line about mislabeled findings matches the new wording.
- **F4 (MEDIUM), accept re-entry state.**
  D7 now states the two accept states: pending, written by the verb, and settled, written by the Bouncer before the Approve seam.
  Only a pending accept settles, and a settled one is "settled earlier" at Call entry, including on a resume after a failed Commit seam.
  D1's Call-entry sentence covers a settled accept.
  The Open risks line now points at D7, and the acceptance criteria add the settled record and the Commit-failure resume.
- **F5 (MEDIUM), facts fixture.**
  The criterion is restated over `round-4-facts.md` for a fixture whose rounds 1 to 3 are judged, with the key `resolved` in round 2's ledger and `open` with rounds 1 and 3 in round 3's ledger.
- **F6 (LOW), D7 identity bound.**
  D7's bound now says any shell addressing the run can record the decision, an agent's included.
  It names the refusal precondition as the whole mechanical bound and says no agent stencil names the verbs, as with `lyx loom approve`.
- **F7 (LOW), Auto-mode assumptions.**
  The D8 restatement is dropped, since the question ledger's Q9 already holds it.
  The gating line now states the assumption it carries (a non-spec draft taken as intent), and the straddle line points at D1, D3 and D6 instead of claiming complete coverage.
- **F8 (LOW), budget-block Reason.**
  The requirement moved into D7's `continue` bullet, with an acceptance criterion, and the Open risks line is removed.

Support log: interview turns 8 and 10 note round 1's changes, Q1 is narrowed to entry-time aliases, and Q12 to Q15 record this round's auto-picks.
`## Review rounds` is left to the review gate.

## Deferred

None.
