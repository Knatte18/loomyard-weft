---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/05-convergence-guard.md
    summary: Card 5's fixture enumeration method still misses the dropped-key read on replays - its replay-path ledger check refuses two rows of TestBouncer_CirclingGuard, whose round-2 CIRCLING ledger drops round 1's key alpha, yet the card neither targets that test nor names it in ImpactSummary, and its Intent claims every round-2-or-later ledger a settle reads keeps its previous keys
---

### [BLOCKING] Card 5 breaks two `TestBouncer_CirclingGuard` rows it does not target

**Location:** `_lyx/plan/05-convergence-guard.md`, the `**Edit:**` list, the `**ImpactSummary:**`, and the Intent sentence "every round-2 or later ledger a settle reads keeps its previous round's keys".

**Issue:**
D8 and card 5 run the key check on the replay path too: a CONTINUE or CIRCLING round on disk at Call entry, with no live judge and no escalation record, is checked before `settle`.
`TestBouncer_CirclingGuard` (`internal/shedadapters/bouncer_circling_test.go:287`) drives exactly that path.
`layoutCirclingBouncer` lays out a judged round 2 with a CIRCLING verdict and spawns nothing, the rows that want `Stuck` write no escalation record, and the test then replaces the ledgers with its own.
Two rows drop round 1's key:

- "at the checkpoint with no recurring open key" writes round 1 `alpha` and round 2 `beta` only (lines 310-311);
- "a rising count over new keys" writes round 1 `alpha` and round 2 `beta`, `gamma`, `delta` (lines 339-340).

Under the guard both rounds are refused.
The three judge outputs are archived, and the Call returns a degraded Stuck whose pointer carries a Reason and no ledger path.
So the row's `ptr.Path == ledgerPath(cfg.RunDir, 2)` assertion fails, and so does its check that `focusPath(cfg.RunDir, 3)` exists, since the guard has archived that file.
The card's own claim that every round-2-or-later ledger a settle reads keeps its previous keys is false for these rows.
The test is not under `**Edit:**`, so the implementer cannot fix it inside the card, and card 5's verify run fails.

I checked every other round-2-or-later ledger a Bouncer settle or harvest reads in `internal/shedadapters`, `internal/shedrecipe` and `internal/loomrecipe`, and each keeps its previous round's keys.
They are: `circlingBouncer`, the escalation fixtures, `openGatingLedgerContent`, the shedrecipe `circlingOpenLedger` pair, the carry-over test's `earlier`/`judged` pairs, the empty `ledger: []` bodies, and the CirclingGuard rows "below the checkpoint", "rounds 1 and 3" and "non-gating recurring key".

**Fix:**
Add `internal/shedadapters#TestBouncer_CirclingGuard` to card 5's `**Edit:**` list and name it in `**ImpactSummary:**` beside the carry-over row.
State the new row content in `**Intent:**` so each row keeps its circling outcome under the guard:

- "at the checkpoint with no recurring open key": round 2's ledger holds `alpha` at status `resolved` beside `beta` open;
- "a rising count over new keys": round 2's ledger holds `alpha` at status `resolved` beside `beta`, `gamma` and `delta` open.

`circlingEvidence` counts only a gating key open in this round and an earlier one, so a resolved `alpha` leaves both rows unearned and `Stuck`, as they are today.
Also reword the Intent's "every round-2 or later ledger a settle reads keeps its previous round's keys" so it holds once these rows change.

**Class:** `design`, filed after reading the prior rounds once this review was saved.
This is the third round in a row with a gap in card 5's fixture enumeration: round 1's review read, round 2's open BLOCKING ledger entry, and now the dropped-key read.
Round 2's fixer report says the dropped-key claim was verified by grepping for `BLOCKING`, and that grep cannot find a dropped key.
The method needs to cover every settle path the guard runs on, harvest and replay, for every verdict word, not only CONVERGED.

## Focus directive verdicts

**Directive 1: CONVERGED-settle fixture enumeration.**
Every in-tree Bouncer CONVERGED settle in the three packages reads one of three review writers, and the card covers each of them:

- `internal/shedadapters`: every CONVERGED harvest writes its report through `bouncerReport`.
  These are `TestBouncer_JudgeCall_Verdicts`, the previous-ledger marker table, `PromptReadsFactsNotArtifacts`, `RunError_CONVERGED`, `DebrisIsNotJudged`, the commit, approve and carry-over settle tests, the clear tests including `EndToEndSequence`, the attach-to-live-judge test, and `Cancellation_ParsedVerdictSurvives`.
  The other report writers in that package (`UnreadableReportFile`, `EmptyReportFile`, `WhitespaceOnlyReportFile`) all degrade before any verdict exists.
- `internal/shedrecipe`: every CONVERGED settle comes through `judgeSeamShuttle` over `layoutBouncerRound1Report`.
  These are the commit-seam, carry-over, approve-seam, cluster-excludes and parent-name tests.
  `TestBouncerEntry_ReportNamePinning` writes its own `a review` body, but its default fake shuttle writes no verdict, so it never settles.
- `internal/loomrecipe`: every CONVERGED settle reads the review `newLoomBurler` writes, and no test installs another Burler runner.

`ParseReview` rejects all three current bodies, since none carries a `verdict:` key.
The card's planned body (`verdict: APPROVED`, `findings: []`) parses, and the tolerated extra `round:` key does no harm.
On the ledger side, every CONVERGED ledger is empty except the carry-over test's first row, and that row is the only open BLOCKING entry under CONVERGED.
The card targets it and names it in ImpactSummary.
No CONVERGED fixture outside the three packages exists, by a repo-wide grep.
For the CONVERGED enumeration the card is complete: not a defect.

**Directive 2: changed row content stated in Intent.**
For the carry-over row the card states `beta` becomes severity `MEDIUM` both in the judged ledger and in the expected carry-over finding.
That keeps the row consistent with the guard and with the MEDIUM-or-worse filter, so `beta` is still carried and the sort order is unchanged: not a defect.
The CirclingGuard rows F1 adds need the same statement, which F1's fix gives.

## Focus departures

- Directive: "Enumerate every test fixture ... that reaches a Bouncer CONVERGED settle".
  F1 concerns a CIRCLING replay, not a CONVERGED settle.
  I followed the guard's ledger read beyond CONVERGED because card 5's own Intent claims every round-2-or-later ledger a settle reads keeps its previous keys, and the rubric requires a finding where that claim breaks a test the card does not target.
