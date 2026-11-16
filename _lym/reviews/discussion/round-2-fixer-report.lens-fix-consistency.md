```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the changed text still agrees with the decisions, criteria and ledger entries it touches.

Findings:

1. **MEDIUM**: D11's new "Every shared refusal names the active implementer's verbs in its way forward" has nothing to name under Weaver.
   - The shared plan-partition refusal names `lym webster rebaseline --card NN` and `lym webster restore-plan` (`internal/websterengine/partition.go:131`, quoted in the review).
   - D2's bound still says Weaver "adds one verb, `lym weaver reset`", so Weaver has no rebaseline or restore-plan verb to name.
   - The D11 rule and the D2 bound disagree. Either D11 says which Weaver step replaces each Webster verb (for example `lym weaver reset`, as D7 names), or D2 adds the verbs.

2. **MEDIUM**: D7's new "maps the done and stopped batches onto the current plan" contradicts D11's new "its recorded partition mapped onto the current plan".
   - D7 maps only the done and stopped batches.
   - Under the review's cited `mapPartition`, any plan card that no mapped batch holds is refused. The pending cards would be such cards, so the re-entry would refuse.
   - That also breaks the new D7 criterion "also after the arbiter edits a pending card".
   - D11 maps the whole recorded partition, pending batches included. The two sentences should agree.

3. **LOW**: Support log, Turn 4 bullet "interactions (MEDIUM): a relaunched gated strand could be attached before its finished report is recorded — fixed: D12 records a finished report first".
   - The F1 change to D12 reverses that design: Weaver now attaches first and records a report only from the Wait result, or directly only when the attach finds no run.
   - The log still describes the reversed design as the fix in force.

4. **LOW**: Support log, Q6 ("pass records done, fail gets one last-rung attempt") is stale against the new D7.
   - It leaves out three things the new D7 adds: re-entry after a `NEEDS_PARENT` stop resumes at the rung that asked, a batch whose last attempt failed on a row gates entry never takes the no-spawn path, and the plan re-read with its `lym weaver reset` refusal.
   - The auto-mode assumptions line got the matching update; Q6 did not.

5. **LOW**: Support log, the new Turn 2 bullet ends "(evidence for Q12)", but Q12 is about the D13 seam ("How do loom's readers of Webster state see Weaver's?").
   - The bullet's own text says it supports D11's move of the state types.
   - No ledger question covers D11, so the cross-reference points at the wrong decision.

No further tool calls made.
