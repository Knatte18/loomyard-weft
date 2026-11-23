```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each proposal the fixer carried out as proposed (F13, F2, F11, F12).

Findings:

1. **LOW**: `internal/configreg/configreg.go:164` with `FlagCodeAutoAccept` (`RunsWithoutFile: true`, absent file flagged).
   - A hub created after this change, with no `loom.yaml`, now mails the orch a "changed what a value means" notice about a change it never lived through.
   - Evidence: `applyReinterpretations` (`internal/hubreconcile/reinterpret.go:64-81`) runs `Rewrite(nil)` for an absent file when `RunsWithoutFile` is set. The flag line then goes to `notifyReinterpretations`.
   - The marker makes it one-shot per hub, so it is noise, not a loop.
   - The orch precedent the review cited (`FlagRemoteControlDefault`) does not set `RunsWithoutFile`. The absent-file behaviour is new here, and the reviewer asked for it.

No other findings. The cases I tried:

- **F13 outside the window.** The Stuck leaves the accept decision file unsettled. A `CirclingContinue` pending on a decision escalation does the same.
  - `settledGeneration` and `standsAsRecorded` (`bouncer.go:916` and `:1102`) treat the round the same either way, so nothing new breaks.
  - Dropping `BudgetExempt` is correct. The window is `budgetReached || atHardCap`, so `budgetReached` is always false in that branch.
- **F13 inside the window.** `retireContinue` moves the escalation and decision files aside, so `escalateWith` writes a fresh budget record and does not replay the old decision-cause one.
  - A later parent accept carries cause budget, so `routeDecisions` returns handled=false and the existing budget path settles it.
  - The new `AutoAccept` row at `spentBounces` exercises this path and passed.
- **F2.** `yaml.Unmarshal(nil)` gives an empty map, and the absent file produces no write since `bytes.Equal(nil, nil)` holds. A file with `code_auto_accept: false` stays unflagged.
- **F11.** The `OtherRowRefusesIt` refusal text `does not list` can only come from the focus exemption missing. The two validate rows differ only in run directory, so each isolates the flag's wiring.
- **F12.** The added `wantDetailAlso` fragment is the exact text built at `bouncerdecision.go:153`.

Tool calls:
1. `grep RunsWithoutFile` and `FlagRemoteControlDefault`: to see whether the orch precedent uses `RunsWithoutFile`. My context lacked orch's file.
2. `Read internal/hubreconcile/reinterpret.go:25-115`: to see how the walk treats a nil file and its marker. My context lacked the walk.
3. `grep decisionEscalationOpen`: to find who reads a pending accept's state. My context lacked the callers.
4. `sed bouncer.go:905-925, 1092-1112`: to see how those callers treat an unsettled accept. My context lacked those spans.
