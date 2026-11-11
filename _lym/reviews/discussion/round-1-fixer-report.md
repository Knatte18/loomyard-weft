# Fixer report — discussion round 1

## Changes

All changes are in `_lym/discussion/decision-record.md` unless named otherwise.

- F1 (BLOCKING): fixed as proposed.
  D2 drops the false generation parenthetical and states the code's facts with evidence:
  `generation-record.json` `Number`, a missing record as 1, `startNextGeneration` restarting round numbers, the clear, goto and resume, and the run status `history`.
  The asking rule now keys on the round's seed, the round-1 focus file the Bouncer's seed wrote,
  and on a `Discussion-Write` history entry after the latest ask, so a goto back to Discussion-Write sends the rewritten record.
  Criterion added for the goto after a satisfied answer.
- F2 (MEDIUM): fixed as proposed, by the seed identity the fix key recommends.
  D3 matches the ask by seed and round, and keeps the ask record outside the run directory; a clear writes a new seed, so it asks again.
  Criterion added for a clear after a `done` ask.
- F3 (MEDIUM): fixed as proposed.
  D5 states the `CapAssistTimeout` formula with a 5-minute consolidation margin, the chair timeout's `run_timeout_min` fallback, and the no-room outcome `not-sent` with one Warn.
  Criterion added; ledger entries added for the margin and the no-room pick.
- F4 (MEDIUM): fixed as proposed.
  D3 maps the nil-error `posted` false and a post error to `no-parent`, and a reached dead orch to `timeout`; the criterion is reworded and a `timeout` criterion added.
- F5 (MEDIUM): fixed differently.
  The request line and the verb carry an ask number (`--ask`, required) instead of generation and round, because generation and round do not identify an ask once a clear re-seeds round 1 under the same generation (F2).
  The verb refuses a mismatched number naming `lym loom decision add`.
  Go closes an open ask when the table ends with its review written, not on failure as proposed:
  `Discussion-Burler-Review` is reinvoked on failure (`internal/loomshed/interruptpolicy.go:47`), and closing the ask then would drop the restart's wait.
  The answer verb closes an ask past its deadline instead. Criteria added for both.
- F6 (MEDIUM): fixed as proposed.
  D4 defines a YAML `findings` list, each item with a required `change` and optional `severity` and `class`, with refusals; D6 points the fork at it; the D7 criterion reads given severity and class.
- F7 (MEDIUM): fixed by the review's named alternative.
  D3 posts through a variant signed as the program `LOOM` whatever strand claim the process carries (`internal/orchcli/notify.go:188-218`), and D6 keys on `<shortname>:LOOM`.
  The first-named form, keying on the run's own sessions, still missed a driver outside the run (self-check finding).
- F8 (LOW): fixed as proposed; D2 says the key decides only a new ask, and a written ask, `pending-post` included, is posted, awaited and consolidated. Criterion added.
- F9 (LOW): fixed as proposed; D3 orders `pending-post`, post, `open`, and states the restart outcome.
- F10 (LOW): fixed as proposed; the ask record stores the strand, resolved before the `pending-post` write, and D4's guard admits it.
- F11 (LOW): fixed as proposed; D4 and D6 bounds state the unverified strand check and the fork's full orch reach.
- F12 (LOW): fixed; the D4 refusal criterion covers `no-parent`, `not-sent` and a mismatched `--ask`.
- F13 (LOW): fixed; D5 criterion for the `timeout` review line.
- F14 (NIT): fixed as proposed; the sentence is cut from D5 and `support-log.md` gains the Rejected alternatives bullet.
- `support-log.md`: Round 1 entry under Review rounds, a Rejected alternatives bullet for keying by generation and round, and updated ledger entries.

## Self-check

- fix-consistency, LOW, `pending-post` ask and the key flip: fixed as proposed (D2 and its criterion).
- fix-way-forward, LOW, `not-sent` has no way forward: fixed with own change; a later round of the same seed asks again after `not-sent`, and the Warn says to raise `review_timeout_min`.
- fix-way-forward, LOW, no stored strand while `pending-post`: fixed as proposed; the strand is resolved and stored before the post.
- fix-claims, MEDIUM, fork reach on "any run": fixed as proposed; "every run whose parent resolves to the orch".
- fix-claims, LOW, history is not one entry per call: fixed as proposed; folding and goto entries stated, the rule reads `At` or `LastAt`.
- fix-claims, LOW, goto halt stated for every unsettled round: fixed as proposed; narrowed to an undecided escalated round, criterion too.
- fix-proposal, MEDIUM, closing the ask on a failed table breaks the restart: fixed as proposed; closed only on a written review, and by the answer verb past the deadline.
- fix-proposal, LOW, D6 key misses drivers outside the run: fixed with the program-signed post (see F7).

## Disputed

None.

## Deferred

None.

## Rulings

None.
