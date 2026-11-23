```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the board entry's goal points and checked each goal point for a change that builds it.

**Decisions.** The answer key is the decision record alone. It has no `Added after Discussion` entries.
- **G1.** In the `## Regression check` block, the line "Baseline: captured at &lt;sha&gt;, the task's branch point." is lym's internal process wording in text the repository reader sees. It should read neutrally, "such as 'the base of this branch'" (`_lym/discussion/decision-record.md:6`).

**Per change:**
- **C1** — `internal/driftengine/verdict.go:58`: the baseline line now reads "the base of this branch." Traces to G1, using exactly the wording the entry asks for.
- **C2** — `internal/driftengine/verdict.go:100,102`: the off-scope line changes from "the task is not regression-checked" to "this change is not regression-checked". Traces to G1's stated defect, internal process wording in the same repository-facing block. The entry's "such as" names an example, so this sits inside the complaint rather than being unrequested scope. The commit message and summary give that reason (`summary.md:16`).
- **C3** — `internal/driftengine/verdict_test.go:45,147`: the expectations are updated to the new strings. Forced by C1 and C2.

**Per decision:**
- G1 is built by C1 and C2. C3 is forced by them.

**Findings:**
1. **NIT, scope:** `internal/driftengine/verdict.go:152`.
   - The line "- Amendments by the run's parent:" stays in the same repository-facing block. It is lym run-model wording ("the run's parent") of the kind G1 complains about, and the diff applies G1's general point to the off-scope line in C2 but not here.
   - The summary keeps it on purpose (`_lym/landing/summary.md:17`): "it names who changed the expectation, which a reviewer needs". The entry's explicit expectation covers only the baseline line, so this is not required. If the handler reads G1 as covering the whole block, the fix is neutral wording that keeps who amended, e.g. "Amendments to the expected changes:" with each entry's `by`.
   - Evidence: `verdict.go:152`, `decision-record.md:6`.

No other scope findings: every hunk traces to G1 or is forced by a change that does, and no `Pattern-Departure:` trailer applies.

Tool calls: none.
