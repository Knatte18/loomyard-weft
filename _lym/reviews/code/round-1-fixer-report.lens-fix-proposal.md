```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1, F2, F4, F6–F10, F3/F5 wording).

**Findings**

1. **LOW** — `escalateUndecided` in `internal/shedadapters/bouncerdecision.go` (F1).
   - Summary: the new budget route also catches a judged CONVERGED verdict that has an open BLOCKING non-decision ledger entry, which the arbiter previously met through the refusal check.
   - Evidence: `routeDecisions` receives `verdict` and passes it on, but the new `softCapWindowOpen` / `blockingEscalationDetail(round, ClassDecision)` check ignores it.
   - Case: round 2 in the window, verdict CONVERGED, ledger `hard` open BLOCKING class scope, one decision finding. Before the fix this escalated with cause `decision`, and `decisionRefusal` showed the "ledger key open at BLOCKING" refusal. Now it escalates with cause `budget` and the detail says "the spent budget leaves BLOCKING findings open". The parent is told about a budget when the real defect is a contradictory judge verdict.
   - Impact: no wrong accept (the parent still rules), only a misleading cause and detail.
   - Not tested: `TestBouncer_AutoAccept` has only CONTINUE rows for this path.

2. **LOW** — the `escalateUndecided` doc comment (F1).
   - Summary: "the detail names findings, the review's decision-class findings" is garbled.
   - Evidence: the doc line "...and the detail names findings, the review's decision-class findings." Rewrite it to name the decision findings directly.

3. **LOW** — `FlagCodeAutoAccept` in `internal/loomengine/reinterpret.go` (F2).
   - Summary: the flag fires on every explicit `code_auto_accept: true`, including hubs whose recipe rows never set `blocking_escalates`. Only the shipped recipes set it.
   - Evidence: the changed behaviour lives in the recipe key `blocking_escalates`, not in the setting. The flag text says "no longer accepts" unconditionally.
   - Case: a hub with a customised recipe that omits the key is flagged for a change that does not affect it.
   - Impact: slightly overstated notice; the reviewer's proposal asked for exactly this wording.

**Challenged and held (no finding)**
- F4: the CLI gate holds.
  - `hardenwiring.go:370-388` builds the campaign run dir as `filepath.Join(LoomReviewsDir(location), subdir)`, the same path `validate.go` compares against.
  - `entries_burler.go` sets `FocusFindings` from `hardenRound`, which both harden review rows set.
  - A non-incremental (amendment) round skips `newFindings` entirely, so the gate loses nothing there.
- F1 follow-through: after a budget escalation is recorded, `routeDecisions` returns `handled=false` for its cause (`escalated && cause != EscalationDecision`). A parent's accept or continue is then settled by the existing `settleUnconverged` path, as the review required.
- F6–F8, F10: the test changes assert what their names state, and nothing else in the context breaks.

**Tool calls**
1. `sed -n 370,395p internal/loomcli/hardenwiring.go` and `grep -n RunRoot` on it: to confirm the hardener's run directory equals `LoomReviewsDir/<subdir>`, which my context lacked.
