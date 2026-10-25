```yaml
lens: fix-proposal
outcome: reported
```

Scope: I challenged each proposal the fixer carried out (F1–F4) by looking for a concrete case where it breaks something.

**[LOW] Card 12's "What bounds it" paragraph leaves out one re-escalation path.**
- **Changed text:** `_lyx/plan/12-bouncer-decision-escalation.md`, the "What bounds it" paragraph: "one escalation per judged round, plus one per decision retired at the budget below the hard cap".
- **Case:** the card's own "A pending `continue`" bullet describes a second way to re-escalate. A `continue` recorded below the cap can meet a lowered `HardCap` or an amendment generation's smaller cap. At or past the hard cap, `retireContinue` then calls `escalateWith(…, EscalationDecision, …)`. That retirement happens at the hard cap, not "at the budget below the hard cap", so the stated bound undercounts the escalations a round can raise. Test row 12 asserts this path: "a `continue` recorded below the cap, settled after `HardCap` is lowered to the round, re-escalates keeping cause `decision`".
- **Evidence:** card 12, the "A pending `continue`" bullet: "at or past the hard cap, `retireContinue` then `escalateWith(…, EscalationDecision, …)`".

I found no break in the other proposals:
- **F1:** the card 6 Prosa group and the card 4 Docs sentence don't conflict with card 17's edits of `docs/overview.md`.
- **F2:** the extra glyphs are needed. Without them, `TestMergeIn_VerifyGate`'s site label and `TestMergeIn_PlanVerifyFailureRecord`'s failure kind would fail once `parentAlignName` and the gate key change. The `strings.Replace` in websterengine's `TestVerifyGate_TellsTheGeometryHoldOnTheFirstRunAndTheRerun` targets only the sentinel command, so changing the gate key there is safe.
- **F4:** `TestConfigTemplate_DecodesAndIsSelfConsistent`'s sentence map is keyed by lens name, not applied to every lens. Dropping the new sentences leaves no generic check that the rewritten texts must still satisfy. The writer-fan containment check (config_test.go:288) still passes, because `goal-scope` and `board-fit` are in `twill-discussion`.

Tool calls:
- One `grep`/`sed -n` over `internal/lenses/config_test.go` and `internal/websterengine/verifygate_integration_test.go`. My context lacked the lens test's sentence-check scope and the restart fixture's `strings.Replace` target.
