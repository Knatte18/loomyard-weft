# Card 5 — convergence-guard

**Create:**
- `plan:internal/shedadapters#Bouncer.judgmentRefusal` -> `func (b *Bouncer) judgmentRefusal(round int, verdict bouncerVerdict) string`
- `plan:internal/shedadapters#Bouncer.rejudge` -> `func (b *Bouncer) rejudge(ctx context.Context, round int, cause string) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/shedadapters#TestBouncer_JudgeCall_RefusedJudgment` -> `func TestBouncer_JudgeCall_RefusedJudgment(t *testing.T)`

**Edit:**
- `internal/shedadapters#Bouncer.Call`
- `internal/shedadapters#Bouncer.judgeCall`

**Prosa:**
- `internal/shedadapters/doc.go`
- `contracts/specs/refusal-spec.md`

**Intent:** Implement D7, D8 and D9: Go refuses a judgment the judge's own rules forbid, and the refused round is re-judged.
`judgmentRefusal` reads only on-disk state under `b.cfg.RunDir` and returns the cause, or empty when the judgment stands.
For a CONVERGED verdict it parses round N's ledger through `parseLedger` and refuses on an entry with status `open` and severity `BLOCKING`, naming the key; it then reads round N's review report at `b.cfg.ReportName(N)` through `burlerengine.ParseReview`, the parser the facts file already uses, and refuses when the report does not parse, naming the error, or carries a finding of severity `BLOCKING`, naming its id.
For any verdict of round N at or above 2, it parses round N-1's ledger; when that ledger is unreadable or does not parse it logs a warning and skips the check, and otherwise refuses when a key of round N-1's ledger is absent from round N's, naming the first missing key; only keys are compared, never status, class or severity.
`rejudge` is the refused judgment's exit: it archives the round's three judge outputs through `archiveStaleOutputs` over `judgeOutputs`, so the round reads as unjudged and the judge stays the only writer of those files, then returns through `degrade` with the Reason `shedadapters: bouncer refused round N's <verdict> judgment: <cause>; way forward: the round is re-judged`; an archive failure degrades naming that failure instead, as `retireLegacyVerdict` does.
`judgeCall` runs the guard between a successful `harvestedVerdict` and `settle`, and so does the entry-time attach branch of `Call`.
On the replay path of `Call`, where a judged round with no live judge reaches `settle` with `spawned` false, the guard runs first unless `readEscalation` reports a record for the round: a round the parent has been handed stands as recorded, which is D7's bound that a pending accept is never refused, and a CONVERGED on disk at entry never reaches this path because the clear precedes it.
The refusal spends one counted bounce unit, returns no `BudgetExempt`, never consults `Bounces` and never escalates, so a run at its budget blocks with Shed's budget-exhausted halt on that refusal with no escalation record and no re-judge.
`TestBouncer_JudgeCall_RefusedJudgment` is one table in `bouncer_judge_test.go`, over the existing judge fixtures and the fake shuttle: CONVERGED with an open BLOCKING ledger entry; CONVERGED over a review with a BLOCKING finding; CONVERGED over an unparseable review; round 2 CONTINUE whose ledger drops a round-1 key; round 2 CONVERGED with a lossless ledger and a malformed round-1 ledger, which settles; a clean CONVERGED, which settles Done exactly as before; CONTINUE with a BLOCKING finding, which passes the guard; a replay of a round-2 CONTINUE on disk with a lossy ledger and no live judge, which is archived and re-judged instead of settled; the same replay with an escalation record on disk, which stays as recorded; and a refusal with the `Bounces` seam reporting the budget reached, which returns Stuck with no escalation record written.
Each refusing row asserts the Stuck Reason names the cause and ends in the way-forward clause, the three judge outputs are archived aside, `BudgetExempt` is false, and no Approve or Commit seam ran.
In `refusal-spec.md`'s shed section, add the row `judgment refused`: the trigger names the four causes and that the three judge outputs are archived, the class is correctness halt, and the Way forward cell quotes `the round is re-judged`.
In `doc.go`, replace the paragraph calling ledger carry-forward a soft spot enforced by the judge prompt alone with a statement of the guard: a CONVERGED with an open BLOCKING ledger entry or a BLOCKING or unparseable review, and any verdict whose ledger drops a previous key, is archived and re-judged within the bounce budget; and extend the Bouncer verdict paragraph with the refusal beside the retired-word route.

**ImpactSummary:** Two new Bouncer methods and three call sites inside Bouncer.Call and judgeCall; a judgment that passes today's parse checks and the new rules settles exactly as before.
