# Card 5 — convergence-guard

**Create:**
- `plan:internal/shedadapters#Bouncer.judgmentRefusal` -> `func (b *Bouncer) judgmentRefusal(round int, verdict bouncerVerdict) string`
- `plan:internal/shedadapters#Bouncer.rejudge` -> `func (b *Bouncer) rejudge(ctx context.Context, round int, cause string) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/shedadapters#TestBouncer_JudgeCall_RefusedJudgment` -> `func TestBouncer_JudgeCall_RefusedJudgment(t *testing.T)`

**Edit:**
- `internal/shedadapters#Bouncer.Call`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/shedadapters#bouncerReport`
- `internal/shedrecipe#layoutBouncerRound1Report`
- `internal/loomrecipe#newLoomBurler`
- `internal/shedadapters#TestBouncer_ConvergedSettle_CallsCarryOverBeforeApproveAndCommit`
- `internal/shedadapters#TestBouncer_CirclingGuard`
- `internal/shedadapters#TestBouncer_Circling_AcceptCallsCarryOverBeforeSettling`

**Prosa:**
- `internal/shedadapters/doc.go`
- `contracts/specs/refusal-spec.md`

**Intent:** Implement D7, D8 and D9: Go refuses a judgment the judge's own rules forbid, and the refused round is re-judged.
`judgmentRefusal` reads only on-disk state under `b.cfg.RunDir` and returns the cause, or empty when the judgment stands.
For a CONVERGED verdict it parses round N's ledger through `parseLedger` and refuses on an entry with status `open` and severity `BLOCKING`, naming the key; it then reads round N's review report at `b.cfg.ReportName(N)` through `burlerengine.ParseReview`, the parser the facts file already uses, and refuses when the report does not parse, naming the error, or carries a finding of severity `BLOCKING`, naming its id.
For any verdict of round N at or above 2, it parses round N-1's ledger; when that ledger is unreadable or does not parse it logs a warning and skips the check, and otherwise refuses when a key of round N-1's ledger is absent from round N's, naming the first missing key; only keys are compared, never status, class or severity.
`rejudge` is the refused judgment's exit: it archives the round's three judge outputs through `archiveStaleOutputs` over `judgeOutputs`, so the round reads as unjudged and the judge stays the only writer of those files, then returns through `degrade` with the Reason `shedadapters: bouncer refused round N's <verdict> judgment: <cause>; way forward: the round is re-judged`; an archive failure degrades naming that failure instead, as `retireLegacyVerdict` does.
`judgeCall` runs the guard between a successful `harvestedVerdict` and `settle`, and so does the entry-time attach branch of `Call`.
On the replay path of `Call`, where a judged round with no live judge reaches `settle` with `spawned` false, the guard runs first unless `readEscalation` reports a record for the round or `readCirclingDecision` reports a decision file for it: a round the parent has been handed or has decided stands as recorded, which is D7's bound that a pending accept is never refused, and a CONVERGED on disk at entry never reaches this path because the clear precedes it.
The decision file counts on its own because `RecordCirclingDecision` records a decision over a legacy CIRCLING round with no escalation record.
The refusal spends one counted bounce unit, returns no `BudgetExempt`, never consults `Bounces` and never escalates, so a run at its budget blocks with Shed's budget-exhausted halt on that refusal with no escalation record and no re-judge.
`TestBouncer_JudgeCall_RefusedJudgment` is one table in `bouncer_judge_test.go`, over the existing judge fixtures and the fake shuttle: CONVERGED with an open BLOCKING ledger entry; CONVERGED over a review with a BLOCKING finding; CONVERGED over an unparseable review; round 2 CONTINUE whose ledger drops a round-1 key; round 2 CONVERGED with a lossless ledger and a malformed round-1 ledger, which settles; CONTINUE with a BLOCKING finding, which passes the guard; a replay of a round-2 CONTINUE on disk with a lossy ledger and no live judge, which is archived and re-judged instead of settled; the same replay with an escalation record on disk, which stays as recorded; a replay of a round-2 CIRCLING with a lossy ledger, no escalation record and a pending accept recorded, which settles Done; and a refusal with the `Bounces` seam reporting the budget reached, which returns Stuck with no escalation record written.
Each refusing row asserts the Stuck Reason names the cause and ends in the way-forward clause, the three judge outputs are archived aside, `BudgetExempt` is false, and no Approve or Commit seam ran.
A clean CONVERGED settling Done exactly as before is not a row of the new table: the `CONVERGED is Done` row of `TestBouncer_JudgeCall_Verdicts` already pins it, and over the fixed `bouncerReport` below it is the guard's pass-through case.
The guard newly reads the review, and every existing review fixture a CONVERGED settle reads carries no `verdict:` key, so `burlerengine.ParseReview` rejects it and each of those settles would be refused until the bounce budget blocks.
Each such fixture therefore renders a review that parses with no BLOCKING finding, a frontmatter holding `verdict: APPROVED` and `findings: []`: `bouncerReport`, which every shedadapters judge, clear, commit, replay, verdict, escalation and attach-probe test reads; `layoutBouncerRound1Report`, which the shedrecipe commit-seam, carry-over and approve-seam tests settle CONVERGED over; and `newLoomBurler`, whose review every loomrecipe sequence test's CONVERGED judge settles over.
The guard's ledger reads break one more settle: the first row of `TestBouncer_ConvergedSettle_CallsCarryOverBeforeApproveAndCommit` settles CONVERGED over a judged ledger holding `beta` open at severity `BLOCKING`, which the guard refuses, so that row's `beta` entry becomes severity `MEDIUM` in the judged ledger and in the expected carry-over finding, keeping the row's label and rounds coverage.
The carry-over of an open BLOCKING entry stays reachable through a pending accept, which D7 lets reach Done over open BLOCKING findings, so `TestBouncer_Circling_AcceptCallsCarryOverBeforeSettling` takes over that coverage: it builds its run directory through `layoutCirclingBouncer` instead of `circlingBouncer`, with round 1's ledger holding `alpha` open at `design`/`MEDIUM` and round 2's holding that `alpha` beside `beta` open at `design`/`BLOCKING`, and its expected carry-over gains the finding `beta`, class `design`, severity `BLOCKING`, rounds `[2]`, after `alpha`.
Round 2 keeps every round-1 key, and the accept replay is exempt from the guard under its decision file, so the test's outcome is unchanged.
The guard's dropped-key read also runs on the replay path, and two rows of `TestBouncer_CirclingGuard` replay a round-2 CIRCLING with no escalation record over a ledger that drops round 1's `alpha`, which the guard refuses.
In each, round 2's ledger keeps `alpha` at status `resolved`: "at the checkpoint with no recurring open key" writes `alpha` resolved beside `beta` open, and "a rising count over new keys" writes `alpha` resolved beside `beta`, `gamma` and `delta` open.
Circling evidence counts only a gating key open in this round and an earlier one, so both rows stay unearned and settle Stuck as today, with the same pointer and next-round focus file.
With those rows changed, every other CONVERGED settle in `internal/shedadapters`, `internal/shedrecipe` and `internal/loomrecipe` carries no open BLOCKING ledger entry, and every round-2 or later ledger a harvest or replay of any verdict reads keeps its previous round's keys.
In `refusal-spec.md`'s shed section, add the row `judgment refused`: the trigger names the four causes and that the three judge outputs are archived, the class is correctness halt, and the Way forward cell quotes `the round is re-judged`.
In `doc.go`, replace the paragraph calling ledger carry-forward a soft spot enforced by the judge prompt alone with a statement of the guard: a CONVERGED with an open BLOCKING ledger entry or a BLOCKING or unparseable review, and any verdict whose ledger drops a previous key, is archived and re-judged within the bounce budget; and extend the Bouncer verdict paragraph with the refusal beside the retired-word route.

**ImpactSummary:** Two new Bouncer methods and three call sites inside Bouncer.Call and judgeCall; the guard's review read rejects the verdict-less review fixtures of shedadapters, shedrecipe and loomrecipe that every in-tree CONVERGED settle reads, so this card fixes those three fixture helpers; the guard's ledger read rejects the open BLOCKING entry of the first row of shedadapters' `TestBouncer_ConvergedSettle_CallsCarryOverBeforeApproveAndCommit`, which this card lowers to MEDIUM while `TestBouncer_Circling_AcceptCallsCarryOverBeforeSettling` gains an open BLOCKING entry so the BLOCKING carry-over stays pinned; its dropped-key read rejects the two `TestBouncer_CirclingGuard` rows whose round-2 ledger drops `alpha`, which this card keeps as resolved; and a judgment over a parsing review that passes the new rules settles exactly as before.
