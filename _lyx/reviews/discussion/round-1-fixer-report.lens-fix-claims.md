```yaml
lens: fix-claims
outcome: reported
```

Task: fix-claims. I checked that the facts the fix added or rewrote hold against the code at HEAD.

Claims checked, all hold:
- **D5, hub-tuned lenses.** A hub- or worktree-defined `goal-scope` or `board-fit` beats the template, and reconcile reports nothing for a name the template holds. `internal/lenses/doc.go` lines 8–11 and 21 say only names the template lacks are reported.
- **D5, `discussion` fan.** No default config names the `discussion` fan. `internal/loomengine/template.yaml` names only `twill-discussion` and `review-discussion`, and darn's template names neither.
- **D6, board titles.** `board-fit` can name decision notes from `lyx board list` titles. The list columns include kind and title (`internal/boardcli/cli.go:279`).
- **D8, settled accept.** "A settled circling accept is a settled generation today" holds (`settledGeneration`, `bouncer.go:833-843`).
- **D8, re-judge.** After a re-judge the round escalates `decision` again. `rejudge` archives only the verdict, ledger and next focus (`round.go:91-97`), so the review and its `decision` finding stay.
- **D8, carry-over.** "Open findings carried into Open risks" matches the carry-over seam's contract (`settle` doc comment).
- **D10, no parent.** A held turn end with no parent reaches nobody until the timeout. A runner with no notifier holds silently to its deadline (`internal/shuttleengine/doc.go:89-91`).
- **D11, record name.** The record is named `Merge-In-fixer-attempts.json`. `mergeInName = "Merge-In"` (`landingshed/mergein.go:18`) is passed to `FixerAttemptsPath` (`mergein.go:208`), and Merge-In removes that record there, so "removes the old record wherever it removes its own" has a real site.

Findings:

1. **LOW.** D8's second bullet, "A judgment the refusal rejects is not passed: the accept retires … and the round is re-judged; the re-judged round escalates `decision` again".
   D8 leaves out that each such re-judge costs a bounce unit, and that an accept over a CONVERGED round whose review keeps a BLOCKING finding can never pass.
   - Evidence: `rejudge`'s doc says its Stuck "spends one counted bounce unit, is never BudgetExempt and never escalates, so a segment at its budget halts on it" (`bouncer.go`, `rejudge`).
   - `judgmentRefusal` refuses every CONVERGED judgment over a review with a BLOCKING finding, and the review survives the re-judge.
   - So repeated accepts on such a round halt the segment at its budget. The "What bounds it" line names only the hard cap and one escalation per judged round, not this budget cost.

Tool calls beyond my context:
- Darn template and Merge-In row constant: to check the `discussion`-fan default and the attempt-record name.
- `mergein.go` removal site and the shuttle notifier docs: to check the removal claim and the no-parent silence claim.
- `internal/boardcli/cli.go` grep: to check that `lyx board list` shows note titles.
- A second `cli.go` grep: the first did not show the list columns.
- `judgeOutputs` in `round.go`: to check that the review survives a re-judge.
