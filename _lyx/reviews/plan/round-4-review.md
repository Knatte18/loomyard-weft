---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/00-overview.md (Shared Decision replay-guard-skips-an-escalated-round); _lyx/plan/05-convergence-guard.md Intent
    summary: The replay guard is skipped only on an escalation record, but RecordCirclingDecision also records a decision over a legacy CIRCLING round with no escalation record, so the guard can refuse a pending accept, which D7 forbids
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/05-convergence-guard.md Intent (TestBouncer_ConvergedSettle_CallsCarryOverBeforeApproveAndCommit row change)
    summary: Lowering beta from BLOCKING to MEDIUM leaves no test pinning that an open BLOCKING entry carries over, though the Intent claims the row keeps its filter coverage and D7 lets a pending accept settle over open BLOCKING findings
---

## Focus directive results

The focus directive asked for an enumeration of every fixture in `internal/shedadapters`, `internal/shedrecipe` and `internal/loomrecipe` that reaches the guard's ledger key check, harvest and replay, for every verdict word.
The guard's key check needs a round N ≥ 2 whose round N-1 ledger carries a key.
Every placeholder ledger (`bouncerLedgerContent` in shedadapters, the `ledger: []` bodies in `internal/shedrecipe/entries_bouncer_test.go:480` and `internal/loomrecipe/fixture_test.go:299`) carries no key, so only fixtures built from keyed ledgers can drop one.
Those are `writeEvidenceLedger`, `evidenceLedgerContent`, `openGatingLedgerContent` and shedrecipe's `circlingOpenLedger`; `writeFactsLedger` and the direct `circlingEvidence` table never reach a Bouncer `Call`.
Walking each keyed fixture that reaches a `Call`:

- `circlingBouncer` (`bouncer_circling_test.go:32`), used by the CIRCLING no-decision, recorded-decision, accept carry-over, failed-commit, settled-accept and malformed-decision tests: `alpha` in rounds 1 and 2, lossless.
- `TestBouncer_CirclingGuard` (`bouncer_circling_test.go:283`): "below the checkpoint" (`alpha`/`alpha`) and "non-gating recurring key" (`alpha` LOW in both) are lossless; "open in rounds 1 and 3" is lossless at round 3 (round 2 `alpha` resolved, round 3 `alpha` open); "at the checkpoint with no recurring open key" (`alpha` → `beta`) and "a rising count over new keys" (`alpha` → `beta`, `gamma`, `delta`) drop `alpha` and are the two rows card 5 targets.
- `escalationBouncer` (`bouncer_escalation_test.go:24`), every escalation test: `alpha` in rounds 1 and 2, lossless.
- `TestBouncer_ConvergedSettle_CallsCarryOverBeforeApproveAndCommit` (`bouncer_commit_test.go:104`), a round-2 CONVERGED harvest: row 1 carries `alpha` and `delta` from round 1 into round 2, row 2 has an empty round-1 ledger; both lossless.
- `TestBouncer_Verdict_CirclingReturnsAwaitingAndReplaysWithoutSpawning` and `TestBouncer_Verdict_CirclingWithLiveJudgeAttachesAndHarvests` (`bouncer_verdict_test.go:178`, `:210`), a round-2 CIRCLING harvest, replay and entry-time attach: `alpha` in both rounds, lossless.
- shedrecipe's `layoutBouncerCirclingRounds` with `circlingSeamShuttle` (`entries_bouncer_test.go:897-930`), a round-2 CIRCLING harvest: `alpha` in both rounds, lossless.
- loomrecipe's judge fake writes `ledger: []` for every round, so no round-2-or-later ledger there can drop a key.

So the settle question resolves to yes: the two `TestBouncer_CirclingGuard` rows are the only fixtures whose round-2-or-later ledger drops a previous key, and both are under card 5's `**Edit:**` and in its `**ImpactSummary:**`.
The Intent's closing claim, that every round-2-or-later ledger a harvest or replay reads keeps its previous round's keys once those rows change, holds for every fixture above.
The stated new row content (`alpha` resolved beside the round-2 open keys) keeps both rows unearned, because `circlingEvidence` needs a gating key open in this round and an earlier one, so both still settle Stuck with the same pointer and next-round focus file.
The commit-test row change also states its new content, `beta` at MEDIUM in the judged ledger and in the expected finding; F2 concerns the coverage that change gives up.

Outside the focus, the shedadapters test budget was checked: the package has 158 top-level tests against a row of 159, so card 5's one new test fits and card 7's raise of the row is correctly placed.

### [MEDIUM] Replay guard exemption keys on the escalation record, not on the recorded decision

**Location:** `_lyx/plan/00-overview.md` Shared Decision `replay-guard-skips-an-escalated-round`; `_lyx/plan/05-convergence-guard.md` Intent, the replay-path sentence.

**Issue:** D7 bounds the guard: "A pending escalation accept the parent recorded reaches Done over a CONTINUE or CIRCLING round without the guard, … and the guard never refuses it."
The plan implements that bound as "the guard runs first unless `readEscalation` reports a record for the round".
But a decision file can exist with no escalation record: `RecordCirclingDecision` (`internal/shedadapters/circling.go:46-66`) accepts a round with no escalation record when its recorded verdict is CIRCLING, the legacy case, and `TestBouncer_Circling_RecordedDecision` and `TestBouncer_Circling_AcceptCallsCarryOverBeforeSettling` exercise exactly that state.
On such a round, the replay guard runs over a pending accept, or a continue decision.
A lossy ledger there gets the judgment archived while the decision file stays behind, pointing at a round that now reads as unjudged, which is the "decision pointing at nothing" the Shared Decision's own rationale names as the reason for the exemption.
The in-tree fixtures stay green only because their ledgers are lossless, so no test catches the gap.

**Fix:** Key the replay exemption on the round's recorded decision as well as its escalation record: skip the guard when `readEscalation` reports a record or `readCirclingDecision` reports a file for the round, and reword the Shared Decision and card 5's Intent to say so.
Add a row to `TestBouncer_JudgeCall_RefusedJudgment`: a replay of a round-2 CIRCLING with a lossy ledger, no escalation record and a pending accept recorded, which settles Done.

### [LOW] Lowering beta to MEDIUM drops the only test of BLOCKING carry-over

**Location:** `_lyx/plan/05-convergence-guard.md` Intent, the `TestBouncer_ConvergedSettle_CallsCarryOverBeforeApproveAndCommit` sentence.

**Issue:** `carriesOver` (`internal/shedadapters/bouncercarryover.go:85-90`) admits an entry whose severity is BLOCKING or MEDIUM, each as its own disjunct.
Row 1 of `TestBouncer_ConvergedSettle_CallsCarryOverBeforeApproveAndCommit` (`bouncer_commit_test.go:121`, `:128`) is the only Bouncer test whose carry-over includes a BLOCKING entry; the accept-path test `TestBouncer_Circling_AcceptCallsCarryOverBeforeSettling` carries only `alpha` at MEDIUM.
Card 5 lowers `beta` to MEDIUM and says the row keeps its "filter, label and rounds coverage", but after the change no test pins the BLOCKING disjunct.
That path stays live after this plan: D7 lets a pending accept reach Done "open BLOCKING findings included", and those findings must reach the decision record's carry-over.

**Fix:** Keep the BLOCKING carry-over pinned on the path where it remains reachable.
Card 5 adds `internal/shedadapters#TestBouncer_Circling_AcceptCallsCarryOverBeforeSettling` to `**Edit:**`, gives its fixture's round-2 ledger an open BLOCKING entry beside `alpha` with the matching expected finding, and drops "filter" from the commit-test sentence's coverage claim.
