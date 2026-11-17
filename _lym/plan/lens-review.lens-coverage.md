```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked every decision, acceptance criterion and branch the cards introduce against the card that carries it and the planned test row that reaches it.

**Findings**

1. **MEDIUM — D1 / D6, Card 10 (`roundUnchanged`).** No row covers the D1 criterion that a fixer-written report which is unparseable or has incomplete dispositions makes its round count as changed.
   - D1's acceptance criterion: a report that exhausts the fix check "leaves the next round full and counts as changed".
   - Card 10's Intent carries the branch ("an unparseable or incomplete report"), but its row list stops at "no record, no post-fix id, or whose report carries `spawned: false`".
   - Card 7's builder row covers only the "next round full" half.
   - Fix: add a `TestBouncer_Stalemate` row where `target_id` equals `post_fix_target_id` but the fixer-written report misses a disposition. It must not count toward the stalemate.

2. **MEDIUM — D3, the "Bouncer does not escalate it again" criterion.** No row asserts the Bouncer-side half of this criterion.
   - The criterion has three parts: the review marks the settled held finding `resolved`, does not carry it, and the Bouncer does not escalate it again.
   - Card 5's `TestCheckReviewRound` row covers only the round check.
   - No Card 10 or `TestBouncer_DecisionEscalation` row runs a round N whose only formerly-held finding is `resolved` and uncarried, and checks that `routeDecisions` takes nothing while the still-open, carried one escalates.

3. **MEDIUM — D4 parent-review exemption, Card 7 (`roundParentReportPath`, `Call` setting `ParentReportPath`).** No row reaches the production path that turns the exemption on.
   - `TestBuildRoundContext` has no row for `ParentReportPath`: no ask, an ask with a report, or an unreadable ask or seed giving empty.
   - `TestBurlerReviewProducer_Call_ProfileAndOpts` has no row where `roundParentAsk` answers a non-zero ask and the profile and probe profile carry the report path.
   - If that wiring breaks, every `origin: parent` finding in an incremental round is refused, and only Card 5's hand-built context rows would still pass.

4. **MEDIUM — Card 9 (`noticeWayForward`).** Two branches have no row.
   - At or past the hard cap the clause must drop `continue`. Naming a verb that `circlingVerb` refuses there is a wrong way forward under `PATTERN-refusal-way-forward`.
   - The empty-slug variant is also unreached.
   - Both planned rows sit below the cap with a slug.

5. **MEDIUM — D6 / D5 wiring, Card 12.** Card 12 argues the wiring needs no test: "The entries copy Env fields into configs the tests cannot read back".
   - The rubric treats a criterion argued to hold without a test as a finding.
   - `BurlerHalfDeps.Snapshot`, `DecisionRecordPath`, `BouncerConfig.Snapshot` and `StalemateRounds` all have nil or zero values that silently keep today's behavior (Shared Decision `nil-seams-keep-today`). A mis-wired entry therefore turns off D4, D5 and D6 with every test still green.
   - Read-back is not needed. A row in `internal/shedrecipe/entries_burlerhalves_test.go` could call the built review producer once with a `shedfake.BurlerHalfRunner` and assert that `round-1-snapshot.json` holds a `target_id`. That covers the review-entry wiring.
   - Evidence: `internal/shedrecipe/entries_bouncer_test.go:526-533` shows today's entry tests check only that construction succeeds.

6. **LOW — Card 10 (`stalemated` order).** "false when `StalemateRounds` is 0" is checked before "true when round N's own escalation record holds `stalemate`".
   - A replay after the key is set to 0, of a CONTINUE round below the window with a pending `stalemate` record, would skip the record-holding branch and fall through.
   - That falls short of D6's "A round whose own pending escalation record holds cause `stalemate` escalates with that cause again".
   - No row covers either the order or the own-record-of-another-cause branch ("false when it holds another cause").

7. **LOW — Card 10 (`stalemated`).** No row covers the unreadable-ledger-in-window branch that yields to `softCapRule`'s budget escalation, or a nil `Snapshot` with `StalemateRounds` 2 never escalating.

8. **LOW — Card 7 (`BuildRoundContext`, `recordRoundStart`).** Several incomplete-round and no-changes-file branches have no row:
   - round N-1's review is absent or does not parse;
   - its snapshot record is absent entirely, not just missing `decisions_routed`;
   - a read error other than absence is returned;
   - `recordRoundStart` writes no changes file when round N-1's record has no `target_id` or `ChangedSince` answers false. That is D4's "the BLOCKING rule holds alone" at the producer.

9. **LOW — Card 8.** The "builder error is printed and exits 1" branch has no row in `TestValidateReviewVerb_RoundChecks`.

10. **LOW — Cards 2 and 5.** The rows do not say they assert each refusal's `way forward:` clause. `PATTERN-refusal-way-forward` (the record's Constraints) requires "a test reaches each" with its message asserted.
    - Card 2's gate branch where the review itself fails `ParseCurrentReview` inside `FixerReportGateEntry` is also unreached.

11. **LOW — Card 5 (`locationPath`).** No row covers an absolute location made relative to `worktreeRoot`, or a location cut at a space. The rows cover only `path:12` and `path#Sym`.

12. **LOW — Card 6.** No row asserts that the block names round N-1's review path, its fixer-report path, and the changes file when `ChangesPath` is set. D2 requires all three.

13. **LOW — Card 3.** No row covers a missing target path being skipped in the overlay `ID`, or `Take` skipping the copy when `CacheDir/<id>` already exists.

**Tool calls**
- `grep` of `internal/shuttleengine/gate.go`, `lym quarry` for `TestEngine_Review*`, a `grep` of target resolution in `internal/burlerengine/profile.go`, and a `grep` of the shedrecipe entry tests. My context lacked whether a PassOnCap let-through leaves `GateOutcome.Passed` true (it does, `gate.go:156`), whether the review tests Card 5 edits exist (they do), and the base target paths resolve against (`profile.go:331`, the worktree root).
- `sed` of `internal/shedrecipe/entries_bouncer_test.go:524-560` plus a list of the shedrecipe test files. My context lacked how entry wiring is tested today, which finding 5 cites.
