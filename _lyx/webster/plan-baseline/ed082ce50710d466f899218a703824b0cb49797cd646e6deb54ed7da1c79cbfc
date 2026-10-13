# Card 5 — soft-cap-rule

**Edit:**
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#NewBouncer`
- `internal/shedadapters#Bouncer.settle`
- `internal/shedadapters#Bouncer.settleUnconverged`
- `internal/shedadapters#Bouncer.settleUndecided`
- `internal/shedadapters#circlingDecisionHeader`
- `internal/shedadapters#recordAutoAccept`
- `internal/shedadapters/doc.go`
- `contracts/recipes/loom-recipe.yaml`
- `docs/overview.md`
- `internal/shedadapters#escalationBouncer`
- `internal/shedadapters#TestBouncer_Escalation_AwaitsWithTheRecordAndLeavesItUntouched`
- `internal/shedadapters#TestBouncer_AutoAccept`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#TestBouncerEntry_BlockingEscalatesAndColorSegment`
- `internal/loomrecipe#newEscalationShed`
- `internal/loomrecipe#budgetEscalationBudget`

**Delete:**
- `internal/loomrecipe#TestEscalation_DarnPerchBudgetSpentEscalatesThenAcceptReachesDescribe`
- `internal/loomrecipe#TestEscalation_BudgetContinueGrantsOneRoundThenAccept`

**Create:**
- `plan:internal/loomrecipe#TestEscalation_DarnPerchRuleContinuesThenHardCapAcceptReachesDescribe` -> `func TestEscalation_DarnPerchRuleContinuesThenHardCapAcceptReachesDescribe(t *testing.T)`
- `plan:internal/loomrecipe#TestEscalation_RuleContinuesOneRoundThenHardCapThenAccept` -> `func TestEscalation_RuleContinuesOneRoundThenHardCapThenAccept(t *testing.T)`
- `plan:internal/shedadapters#Bouncer.softCapRule` -> `func (b *Bouncer) softCapRule(round int, verdict bouncerVerdict, ptr shedengine.OutputPointer) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/shedadapters#designWrongKeys` -> `func designWrongKeys(ledger ledgerFile) map[string]bool`
- `plan:internal/shedadapters#readLedger` -> `func readLedger(runDir string, round int) (ledgerFile, error)`
- `plan:internal/shedadapters#recordAutoContinue` -> `func recordAutoContinue(runDir string, round int) error`
- `plan:internal/shedadapters#taggedEntry` -> `type taggedEntry struct`
- `plan:internal/shedadapters#taggedLedgerContent` -> `func taggedLedgerContent(round int, entries ...taggedEntry) string`
- `plan:internal/shedadapters#TestBouncer_SoftCapRule` -> `func TestBouncer_SoftCapRule(t *testing.T)`
- `plan:internal/loomrecipe#designWrongBouncerJudge` -> `func designWrongBouncerJudge(env shedrecipe.Env)`

**Intent:** At a spent review budget the Bouncer settles the round itself from the judge's fix tags (D2), replacing the budget escalation, and the auto-accept perches' replay and `blocking_escalates` paths go to the same rule (D8).

`designWrongKeys(ledger)` returns the keys of the entries that count as design-wrong: `status: open`, severity BLOCKING or MEDIUM, and `Fix` anything but `mechanical`, so a missing or unknown tag counts (D1).
A LOW, NIT or unlabelled-severity entry never counts.
`readLedger(runDir, round)` reads and parses `ledgerPath(runDir, round)`, wrapping either failure with the round.
`recordAutoContinue(runDir, round)` records the round's decision as a pending `continue` with cause `budget` and `automatic: true`, through `createCirclingDecision`.
`circlingDecisionHeader`'s doc says `Automatic` is true for a decision the Bouncer recorded itself: an auto-accept, or the soft-cap rule's accept or continue; `recordAutoAccept`'s doc names both its callers' cases.

`Bouncer.settleUndecided(round, verdict, spawned, ptr)` (no decision file for the round), in order:

1. The soft cap is reached (`budgetReached`), `AutoAccept` is set, the call harvested (`spawned`), the round has no escalation record, and not (`BlockingEscalates` with an open BLOCKING entry or an unreadable ledger, through `openBlocking`): record an automatic accept and `acceptRound`, as today (D8).
2. The window is open (`budgetReached(round)` or `atHardCap(round)`), or the verdict is CIRCLING, and the round's escalation record exists or cannot be read: `b.escalate`, which returns the recorded Awaiting or the unreadable-record Stuck.
   A round that already escalated keeps its record (D2).
3. The window is open: `b.softCapRule(round, verdict, ptr)`.
4. CIRCLING below the window escalates with cause `circling`; a CONTINUE calls `ensureFocus(round + 1)` and returns Stuck, as today.

The window opens on the soft cap by Shed's count or on the hard cap by the round number, whichever comes first, so a round past a goto that reset Shed's count is still settled by the rule (D2, D3).

`Bouncer.softCapRule(round, verdict, ptr)`, in order (D2's steps):

1. `reviewFactsRow(b.cfg.RunDir, round, b.cfg.ReportName).Err` is non-empty, the row the facts file carries as a parse error: escalate with cause `budget`.
   A `readLedger(round)` failure also escalates `budget`.
   That branch is a fail-loud guard with no test row: every caller reached `settle` through `recordedVerdict` or `harvestedVerdict`, which already parsed this ledger.
2. `designWrongKeys` of the round's ledger is empty: `recordAutoAccept`, then `acceptRound`.
   The carry-over goes into `## Open risks` where the recipe sets `carry_over`, the accept is approved and committed, and no escalation record or parent notice is written.
3. `atHardCap(round)`: escalate with cause `hard-cap`.
4. The verdict is CIRCLING: escalate with cause `circling`.
   `settle` has already read an unearned CIRCLING as CONTINUE.
   Otherwise, from round 2, read round N-1's decision file.
   A read error escalates `budget`.
   When it records a `continue`, read round N-1's ledger, where a read or parse error escalates `budget`.
   A key in both rounds' `designWrongKeys` escalates with cause `circling`.
   Only the immediately previous round is consulted, so a key that was LOW, NIT or `mechanical` there does not return.
5. Otherwise `recordAutoContinue(round)`, `ensureFocus(round + 1)`, and return Stuck with the ledger path and `BudgetExempt` true (D4).
   A failed decision write returns an error, as `recordAutoAccept`'s failure does today.

The rule reads only files in the run directory, so a harvest and a replay settle alike on a perch without auto-accept.
A replay of a round the rule settled finds its decision file and takes card 3's recorded-decision path.
A re-call over a rule-made `continue` below the hard cap returns the same exempt Stuck.

Bound (D2, carried into `settleUndecided`'s and `softCapRule`'s doc comments): the rule accepts fixes no reviewer saw only when no open BLOCKING or MEDIUM entry is tagged design-wrong.
It buys rounds past the soft cap one at a time and never at or past the hard cap.
A finding the judge mis-tags `mechanical` lets an unreviewed design change through, bounded only by the soft-cap window, the carry-over into `## Open risks` where the recipe sets it, and the decision file's `automatic` flag.

Doc comments: `BouncerConfig`'s `Bounces` field says a reached budget opens the soft-cap rule's window.
Its `AutoAccept` field says the flag accepts only a harvest at the soft cap, and that a replay and a `BlockingEscalates` round go to the rule.
Its `BlockingEscalates` field says an open BLOCKING entry sends the round to the rule instead of the auto-accept.
`shedrecipe.Env`'s `RowAutoAccept` field doc says a Bouncer told true accepts a harvested round at a spent budget instead of running the soft-cap rule, and that a replay or a `blocking_escalates` round goes to the rule.
`NewBouncer`'s budget-rule paragraph names the rule's automatic continue as exempt.
`settle`'s and `settleUnconverged`'s docs say the undecided path is the rule, not a budget escalation.
`internal/shedadapters/doc.go`'s Escalation and Auto-accept paragraphs describe the window, the five steps, the three causes and the D8 split.
`contracts/recipes/loom-recipe.yaml`'s Merge-Bouncer comment says a BLOCKING finding sends a spent budget to the soft-cap rule even where loom.yaml auto-accepts the round.
`docs/overview.md`'s review-escalation lines (the "for one of two causes" sentence and the "A `continue` after a budget escalation runs exactly one more round" sentence) say instead that a review segment settles a spent budget by the soft-cap rule.
They also say it escalates for a circling design-wrong finding, at the hard cap, or when the rule cannot read its inputs.

Tests in `internal/shedadapters`:

- `taggedEntry` holds `key, status, class, severity, fix`, and `taggedLedgerContent` renders a round's ledger from them as `writeEvidenceLedger` does, writing `fix:` only when set.
  Both sit in `bouncer_escalation_test.go`.
- `escalationBouncer` now overwrites round 2's report at `cfg.ReportName(2)` with `not a review\n`, and its doc says so.
  Its rounds are in the rule's window wherever its callers spend the budget, and an unparseable report keeps their cause `budget` through step 1.
  So `TestBouncer_Escalation_RecordedDecision`, `TestBouncer_Escalation_MissingStencilStillAwaitsWithThePlainReason` and `TestBouncer_Escalation_FailedRecordWriteErrorsNamingTheWayForward` keep their budget escalation unchanged.
  Card 3's hard-cap rows keep their causes, since their pre-written records or step 1 decide them.
- `TestBouncer_Escalation_AwaitsWithTheRecordAndLeavesItUntouched` gains a `parseableReport` field that rewrites round 2's report with `bouncerReport(2)` after `escalationBouncer`.
  Its doc replaces "a spent budget wins over a CIRCLING ruling" with: a round in the rule's window whose review does not parse escalates `budget`, even over a CIRCLING verdict.
  A new row "a design-wrong finding at the hard cap escalates hard-cap" runs CONTINUE, `spentBounces`, hard cap 2 and a parseable report, wanting cause `hard-cap`.
  The fixture's open `alpha` MEDIUM design entry has no `fix` key, so it counts as design-wrong.
  Through the test's existing checks, the row pins the `hard-cap` record, brief and notice, the accept-only Reason and the untouched re-call.
- `TestBouncer_AutoAccept`'s doc says a replay and a `BlockingEscalates` round go to the rule.
  A `wantContinue` field checks the decision file holds a pending `continue` with cause `budget` and `automatic` true; its branch sits before the `else if tt.decide == "" && exists` branch, which keeps failing any other row that leaves a decision file.
  These rows are the rule's continue over an untagged MEDIUM entry on all three paths, which pins that a missing `fix` key reads as design-wrong.
  Rows change as follows:
  - "BlockingEscalates escalates with cause budget on an open BLOCKING entry" becomes "BlockingEscalates sends an open BLOCKING entry to the rule, which continues": Stuck, `wantExempt`, `wantContinue`.
  - "the flag false escalates with cause budget" becomes "the flag false sends the round to the rule, which continues": Stuck, `wantExempt`, `wantContinue`.
  - "a replayed CONTINUE escalates" becomes "a replayed CONTINUE goes to the rule, which continues": Stuck, `wantExempt`, `wantContinue`.
  - The two recorded-parent-decision rows set `escalated: true`, so their first Call still awaits on the pre-written record.
  The other rows stay, among them the harvested auto-accept at the soft cap over an open design-wrong entry, which pins that a non-Merge auto-accept perch still accepts its harvest (D8).
- New `TestBouncer_SoftCapRule` builds its Bouncer as `TestBouncer_AutoAccept` does.
  That is `newBouncerFixture` with `withNestedRunDir`, `withShuttle` and `withEscalationInputs`, `CirclingCheckpoint` 2, `circlingSeams` and `carryOverRecorder`.
  Round 1 is judged CONTINUE over `prevLedger`, and round 2 is judged now by `judgeFakeShuttle` (harvest) or laid out on disk (replay), over `ledger`.
  Optional fields: `prevDecision` writes round 1's decision through `createCirclingDecision` as an automatic budget-cause decision, `badReport` overwrites round 2's report, `badPrevLedger` overwrites round 1's ledger with `not a ledger\n` after layout, and `badPrevDecision` writes `accept\n` as round 1's decision file.
  The bounce seam and `HardCap` are per row, with `spentBounces` (3 of 3 at round 2) unless named.
  That count is the D2 case of a soft cap reached at a round numbered below the budget by counted degrades, so every exempt row also pins D4's exemption there.
  It asserts the outcome and pointer path, `BudgetExempt`, the escalation cause and the Reason's "cause: <cause>", and for `hard-cap` the Reason's lack of `lyx loom circling continue`.
  It asserts no escalation record and an empty `ParentNotice` on non-Awaiting rows, and the decision file: a settled automatic accept with `approve,commit` and one `CarryOverAccepted` entry with `wantCarried` keys, a pending automatic budget `continue`, or none.
  Every row Calls a second time, the replay of a rule-settled round, and asserts the judge spawned no further run by comparing `len(shuttle.Specs)` before and after, since `Called` stays true from the harvest.
  A continue row requires the identical exempt Stuck.
  An Awaiting row requires the identical Awaiting.
  A Done row requires the clear of the settled generation, an archived run directory at `archivedRunDirPath`, with no second `approve,commit`, so the accept is never settled twice.
  Rows, each named in the test, three variants marked `×3` running once on a harvest with no auto-accept, once on a replay with `AutoAccept`, and once harvested with `AutoAccept` and `BlockingEscalates` over a BLOCKING entry in place of the MEDIUM one.
  A `design-wrong` entry below means an open MEDIUM design entry tagged `fix: design-wrong` unless the row says otherwise:
  - no design-wrong entry (an open LOW design-wrong entry and an open MEDIUM `mechanical` entry): Done, automatic accept, carried keys the MEDIUM `mechanical` key alone, since the carry-over keeps only open entries at MEDIUM or worse (`Bouncer.buildCarryOver`);
  - a `design-wrong` entry: continue (its untagged siblings on all three paths are `TestBouncer_AutoAccept`'s rewritten rows);
  - an open MEDIUM entry with `fix: sideways`: continue;
  - an earned CIRCLING (gating key open in rounds 1 and 2, design-wrong): `circling` ×3, plus one at bounces 4 of 3 naming the past-the-soft-cap case of the criterion;
  - previous decision `continue`, the same key design-wrong in both ledgers: `circling` ×3;
  - previous decision `continue`, the key LOW in round 1 and design-wrong now: continue; and `mechanical` in round 1 with the other two variants: continue ×2;
  - hard cap 2 with a design-wrong entry: `hard-cap` ×3; hard cap 2 with none: Done ×3;
  - hard cap 2 with a design-wrong entry and a pre-written `circling` escalation record for round 2: the record's `circling` Awaiting, record unchanged;
  - `belowBounces` (a goto reset Shed's count) with hard cap 2 and CONTINUE: `hard-cap` with a design-wrong entry, Done without;
  - with a design-wrong entry: `badReport`: `budget`; previous decision `continue` with `badPrevLedger`: `budget`; `badPrevDecision`: `budget`;
  - below the window (`belowBounces`, no hard cap) a CONTINUE over a design-wrong entry stays a counted plain Stuck with no decision file.
  A hard cap equal to the soft cap reaches the Bouncer as the hard-cap rows do, since the Bouncer sees only the seam's count, so it is pinned at load by card 6.

Tests elsewhere:

- `TestBouncerEntry_BlockingEscalatesAndColorSegment`'s row "blocking_escalates escalates" becomes "blocking_escalates sends the open BLOCKING entry to the rule, which continues", wanting Stuck.
  Card 4's hard-cap row keeps awaiting.
  `TestBouncerEntry_CirclingReasonSlugAndBudget`'s "SlugAndSpentBudget" still reads cause `budget`: its round-2 review `a review\n` does not parse, so step 1 decides.
- `designWrongBouncerJudge(env)` installs a judge on `env`'s loom shuttle, as `blockBouncerJudge` does.
  The judge writes CONTINUE through `writeBouncerJudge` and then overwrites the ledger with one open entry: key `design-gap`, class `design`, severity MEDIUM, `fix: design-wrong`.
- `budgetEscalationBudget` stays 2, and its doc comment adds that the escalation tests run under a hard cap one above it.
  `newEscalationShed` sets `env.RowHardMaxBounces` to `budgetEscalationBudget + 1` for `Discussion-Bouncer` and installs `designWrongBouncerJudge` instead of `blockBouncerJudge`, and its doc says so.
- Both loomrecipe fixtures run the burler from `newLoomBurler`, whose `DuringReview` writes `---\nverdict: APPROVED\nfindings: []\n---\nreview\n` (`internal/loomrecipe/fixture_test.go:58-63`), a review `burlerengine.ParseReview` accepts, so the rule passes its step 1.
- `TestEscalation_RuleContinuesOneRoundThenHardCapThenAccept` replaces `TestEscalation_BudgetContinueGrantsOneRoundThenAccept`, whose name and assertions pinned the retired budget escalation: the first Run awaits at `Discussion-Bouncer` with "(cause: hard-cap)" and a parent notice after exactly 3 review rounds.
  `round-2-circling-decision.md` holds `decision: continue`, `cause: budget` and `automatic: true`, and `Discussion-Bouncer` has exactly one budget-exempt Stuck.
  Then `RecordCirclingDecision(accept)` returns round 3 with cause `hard-cap`, and the next Run records a `Discussion-Bouncer` Done.
- `TestEscalation_DarnPerchRuleContinuesThenHardCapAcceptReachesDescribe` replaces `TestEscalation_DarnPerchBudgetSpentEscalatesThenAcceptReachesDescribe`, keeping its Describe and no-decision-record checks: it installs `designWrongBouncerJudge` and sets `env.RowHardMaxBounces` to 2 for `Darn-Bouncer` under darn's budget of 1.
  The first Run awaits at `Darn-Bouncer` with "(cause: hard-cap)" and the brief path after one automatic continue, and the accept returns cause `hard-cap` and reaches Describe once.
  Both escalation tests' file header says a spent budget now runs the soft-cap rule.

**ImpactSummary:** The Bouncer's undecided-round path for every Bouncer row, loom's four perches and darn's: a spent budget no longer escalates `budget` unless the rule cannot read its inputs; tests that relied on the old budget escalation are rewritten in shedadapters, shedrecipe and loomrecipe, and the BLOCKING auto-accept guard's meaning moves to the rule.

**Commit:** 5: Bouncer settles a spent budget by the soft-cap rule from the judge's fix tags
