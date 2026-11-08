```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals behind F1 (plan-reading test) and F2 (overview wording), both fixed as proposed.

**Findings**

1. **LOW**, `docs/overview.md` regression-rows paragraph, "A hub that leaves the gate off records each of them skipped and spawns or runs nothing for them."
   - The fix put a second gate in the preceding sentence, so "the gate" now reads as ambiguous.
   - Case: a sampler hub with `drift.enabled` on and `drift.scope` left at the default `""` also records every row skipped. The review's holistic check says this ("every row is skipped"). The sentence still attributes skipping only to the off gate.
   - A reader could take "leaves the gate off" to mean `drift.enabled` alone.
   - Possible rewording: "A hub that fails either gate records each of them skipped…".

**F1: no findings.**
- The test would fail with the old `!darnRun` condition. A sampler run would then get `planparser.PlanDir`, so `plan_reading` would contain the plan directory and the `wantHasDir=false` row would fail.
- The reviewer proposed asserting the Drift deps plan directory directly, and the fixer asserts `plan_reading` through `ExpectTable` instead. That is a valid observable proxy: `seats.go:194-197` derives the text only from `PlanDir`, and `"none:"` appears only when it is empty.
- The proposed row set (sampler, darn, tapestry) is covered.

I made no further tool calls; my context already held every file the lens needed.
