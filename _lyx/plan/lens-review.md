# Lens review — judge-cap-call plan

All four forks returned a report.

## criteria-coverage

- MEDIUM, unreadable round ledger untested (card 5): rejected. `softCapRule` is reached only through `settle`, after `recordedVerdict` (`internal/shedadapters/bouncer.go:420`) or `harvestedVerdict` (`bouncer.go:356`, `bouncer.go:1271`) parsed this very ledger, so no test input reaches a failing re-read. The branch is kept as D2 step 1's fail-loud guard, which takes no row.
- MEDIUM, unreadable previous decision file escalating `budget` is unlicensed scope (card 5): rejected. Step 4 must read round N-1's decision file to apply D2's "after a continue" test, and `readCirclingFile` returns an error for a malformed file (`internal/shedadapters/circling.go:176-200`). The record's Goal keeps an escalation "where the rule cannot read its inputs", which this is. The alternative, reading a malformed file as "no continue", would silently skip D2's circling check.
- LOW, hard cap with no design-wrong entry accepted on one path only: fixed, Done ×3.
- LOW, past-the-soft-cap CIRCLING on one path only: rejected. `budgetReached` compares `count >= budget` alone (`bouncer.go:916`), so "past" takes the same branch as "at" on every path. One named row documents it.
- LOW, a kept escalation record at the hard cap unpinned: fixed, new row with a pre-written `circling` record at hard cap 2.
- LOW, replay of a rule-made accept unpinned: fixed, every row now re-calls, and a Done row requires the settled-generation clear with no second approve or commit.
- LOW, D9's accept and raise-the-cap way forward unclaimed: fixed, two scoped orch claims in card 9.
- LOW, brief's new meaning of `budget` unclaimed: fixed, claim "could not read its inputs" in card 3.
- LOW, the loomrecipe test names now lie: fixed, card 5 deletes both and creates `TestEscalation_RuleContinuesOneRoundThenHardCapThenAccept` and `TestEscalation_DarnPerchRuleContinuesThenHardCapAcceptReachesDescribe`.

## row-reachability

- BLOCKING, `wantCarried` holding the LOW key (card 5): fixed. The carry-over keeps only open entries at MEDIUM or worse (`Bouncer.buildCarryOver`), so the row wants the MEDIUM `mechanical` key alone.
- MEDIUM, sequence-fixture reviews may not parse, so the loomrecipe tests would stop at step 1 (card 5): rejected. Both fixtures use `newLoomBurler` (`internal/loomrecipe/shape_test.go:103`), whose `DuringReview` writes `---\nverdict: APPROVED\nfindings: []\n---\nreview\n` (`internal/loomrecipe/fixture_test.go:58-63`), a review `ParseReview` accepts. The card now cites it.
- LOW, the `badPrevDecision` and `badPrevLedger` rows name no design-wrong entry, so step 2 accepts first: fixed.
- LOW, the "no new judge spawn" check is vacuous on `Called`: fixed, it compares `len(shuttle.Specs)`.
- LOW, `TestBouncer_AutoAccept`'s `else if tt.decide == "" && exists` branch fails the new continue rows: fixed, the `wantContinue` branch sits before it.
- LOW, `seedDarnHubConfig` not listed: fixed, `Uses` lists it and `hubLocation`.

## forced-edits

- LOW, stale "reaches the budget escalation" comments in loom's template, `LoadConfig` and the `TestLoadConfig_Loads` doc: fixed in card 6.
- LOW, `config.go`'s header still says Merge-Review escalates on an open BLOCKING: fixed in card 6.
- LOW, `TestBouncer_Escalation_RecordedDecision`'s doc and `//testtiming:keep` reason: fixed in card 3.
- LOW, `TestBouncer_Escalation_AwaitsWithTheRecordAndLeavesItUntouched`'s doc "a spent budget wins over a CIRCLING ruling": fixed in card 5.
- LOW, `escalation.go`'s header says "either kind": fixed in card 2.
- LOW, loomrecipe test names: fixed (merged with the criteria-coverage finding).
- LOW, the `Env.RowAutoAccept` doc: fixed in card 5, which lists `internal/shedrecipe#Env`.

## test-economy

- LOW, SoftCapRule's untagged and ×3 continue rows repeat `TestBouncer_AutoAccept`'s rewritten rows: fixed, SoftCapRule keeps one tagged row and points at the AutoAccept rows.
- LOW, the equal-caps Bouncer row repeats the hard-cap branch: fixed, dropped there, and card 6 adds a `TestLoadConfig_Loads` row for equal caps.
- LOW, the 4-of-3 CIRCLING row repeats 3-of-3: rejected, it names the criterion's "past the soft cap" case in one row.
- LOW, card 8's past-the-cap and run_settings refusal rows repeat the at-the-cap branch: past-the-cap row fixed (dropped). The run_settings row is rejected: it pins the record's criterion that the verb's refusal carries the local-layer route.
- LOW, card 8's `PR-Gate` guard row: fixed, dropped, since the branch is a wiring guard.
- LOW, two known-value `fix` rows in card 1: fixed, the `design-wrong` row is dropped.
