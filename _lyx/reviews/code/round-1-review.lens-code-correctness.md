```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over the Bouncer `decision` escalation, carry-over, arbiterstop and the Parent-Align attempt-record carry-across. I found no defects.

**List (function or branch, case run, verdict)**

- `routeDecisions`, `EscalateDecisions` off: returns unhandled, so every other row behaves as before. Correct.
- `routeDecisions`, review unreadable or unparseable: returns unhandled and falls through to `judgmentRefusal`, which reports the parse failure. Correct.
- `routeDecisions`, no decision finding: unhandled. Correct.
- `routeDecisions`, escalation record of another cause, decision file of another cause, or a settled accept: unhandled, so the existing paths run. Correct.
- `routeDecisions`, unreadable decision file with an escalation present: `degrade` with a way forward. Without an escalation it falls through. Correct.
- `routeDecisions` / `escalateUndecided`, first call and CONVERGED re-call (`bouncer.go:498-502`, `standsAsRecorded` is false while the escalation is open): writes the snapshot once, then `escalateWith` returns the same Awaiting from the existing record. Correct.
- `escalateUndecided`, snapshot write failure: returns an error carrying a way forward, and the re-call re-escalates. Correct.
- `settleDecisionContinue`, dropped ledger key: `rejudge` spends one bounce unit and keeps the escalation record and decision file. Correct.
- `settleDecisionContinue`, below the hard cap: `ensureFocus(round+1)`, then Stuck, budget-exempt only when `budgetReached`. This matches `settleUnconverged` for a non-budget cause. Correct.
- `settleDecisionContinue`, at or past the hard cap: `retireContinue` archives the escalation record, parent notice and decision file, so the fresh `escalateWith` writes a new brief with cause `decision`. The snapshot is untouched and is kept. Correct.
- `settleDecisionAccept`, no refusal: `acceptRound` runs. Correct.
- `settleDecisionAccept`, refusal with budget left: `rejudge`. Correct.
- `settleDecisionAccept`, refusal held at the budget below the hard cap: `retireContinue`, then re-escalation with `held=true`. Correct.
- `settleDecisionAccept`, refusal held at the budget at or past the hard cap: archives the judge outputs and returns a `Halt` Stuck whose Reason begins `DecisionHaltPrefix` and names `lyx loom resume`. Correct.
- `decisionRefusal` / `refusalLeavingOut`: a `decision`-class BLOCKING ledger entry or review finding is left out of both checks, and an open BLOCKING entry with no class is still counted. Correct.
- `decisionEscalationOpen`: an unreadable decision file or a pending accept counts as open, and only a settled accept closes it. `settledGeneration` and `standsAsRecorded` use it correctly.
- `escalationReason`: the `decision` cause adds the `lyx loom decision add` step, and the hard-cap wording offers accept only. Correct.
- `renderEscalation` with `FillOptional`: `cause_detail` renders empty for other causes, and a render failure degrades to the plain Reason. Correct.
- `writeDecisionSnapshot`: written only when absent, and an empty path or missing record gives an empty list. Correct.
- `addedSinceSnapshot`: a multiset comparison, and no snapshot gives none. Correct.
- `addDecisionAcceptFindings`: a decision finding at any severity is carried with the titles added since. A ledger entry with the same key takes the titles instead of a duplicate. Other BLOCKING findings of the review are added by key. Correct, with the exact-key matching limit stated in the code comment.
- `renderCarryOver` with `AddedSince`: nil renders the finding bare. Correct.
- `arbiterstop.Check` / `Eligibility`: `DecisionHalt` is set only for a blocked state with the prefix, and the Awaiting branch refuses a `decision` cause. Correct.
- `BurlerFix`: `DecisionsEscalated` sets `DecisionsRouted` and suppresses `sendDecisionFindings`. Correct.
- `loomengine` `retiredLoomKeys`: the four keys are dropped through `DropKeys`, the config field and validation are removed, and no dangling reference shows in the diff. Correct.
- `landingshed` `MigrateConfig` with `retiredKeyRenames`: `merge_fix_attempts` is renamed to `parent_align_fix_attempts` with its value, an existing new key wins, and a second pass is unchanged. Correct.
- `ParentAlign.readAttempts`: reads the retired `Merge-In` attempt record only when its own is absent, and writes `AlignFrom` to its own path. Correct.
- `ParentAlign.removeAttempts`: removes both records. Correct.

**Not checked (outside the tool budget):** the `RenameProducers` status carry-across, the Code-Perch old fixer-report fallback, and the run-profile and `run_settings` read-time migrations.

**Tool calls**
1. `grep` for `retireContinue` and the `escalationPath` / `decisionSnapshotPath` users in `internal/shedadapters`: to see what a retire archives, and that the snapshot survives it.
2. `grep` for the call sites of `settleUnlessRefused`, `standsAsRecorded` and `settledGeneration` in `bouncer.go`: to confirm a recorded decision reaches `routeDecisions`.
3. Read `bouncer.go:405-510`: the Call entry flow.
4. Read `bouncer.go:1100-1250`: `rejudge`, `settle`, `settleUnconverged` and `retireContinue`.
5. Read `landingshed/config.go:170-260` and grep `parentalign.go` for the attempts handling: the migration and the retired-record fallback.
6. Read `parentalign.go:275-355`: `removeAttempts`, `readAttempts` and `align`.
