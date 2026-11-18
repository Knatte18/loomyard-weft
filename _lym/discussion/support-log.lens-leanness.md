```yaml
lens: leanness
outcome: reported
```

Lens: leanness. I looked for content in the decision record that should move out or be cut, applying the completeness-before-leanness test first.

**Findings**

1. **MEDIUM** — `## Auto-mode assumptions`: most bullets repeat what a Decision already says.
   - Evidence: "worktree's own map wins" is in D1; "the prime's `--map` run is the only writer of the index" is in D1; "the trial runs outside this run, as a board note" is in D11; "the changed lines for D9 come from the same base…" is in D9.
   - Only the fixed caps (five tests, 200 commits) and the specificity measure add anything, and D4/D8 state those values too.
   - Fix: keep only what makes them assumptions (that no one asked to tune the caps), cut the rest, and point at the Ds.

2. **MEDIUM** — `## Constraints`: several bullets restate a Decision's own mechanism instead of a constraint.
   - Evidence: the `PATTERN-tolerant-records` bullet repeats D3/D10's "through `internal/state`".
   - The `PATTERN-test-speed` bullet repeats `## PATTERN changes`.
   - The last bullet, about what the maps cannot see, repeats D4 and D7.
   - The `PATTERN-gate-slot` bullet repeats D3, D6 and D10.
   - Fix: keep each bullet as one line naming the entry and any part the Ds leave out, and drop the restatements.

3. **MEDIUM** — `## Notes for the plan writer`, third bullet ("check that each caller's file holds the full failure report block"): this is a deferred research question, not a hint. D7's "no fixer stencil changes for it" depends on the answer.
   - Evidence: the exploration found that `websterengine/verifyfixer.go` points the fixer at `VerifyGateReportPath(...)`. That is a Webster report and not necessarily the gate's stderr failure report.
   - Fix: settle the fact now and record it with evidence in D7, or bound D7 to the callers whose report holds the block. Then drop the note.

4. **LOW** — `## Goal` repeats Scope and the Decisions. The numbered list of three uses is restated in Scope "In" and again in D3, D7 and D9.
   - Fix: keep the build order and cut Goal to one sentence per use, or drop the "In" bullets that only repeat Goal.

5. **LOW** — some rationale is deliberation narrative that belongs in the study or the support log:
   - D7: "all four flaky failures the study could check reached none of their landing's files".
   - D9: "Google's experience…".
   - D4: "153 functions are reached by 500 tests or more".
   - None of these carries a requirement the plan writer acts on. Fix: cut each to a pointer to the report, or move it to the support log.

6. **LOW** — the surviving-mutant output is described twice. D9 says a survivor lists other-package tests as possible killers, and D10's rubric note bullet restates the same listing. Say it once, in D9, and have D10 refer to it.

7. **LOW** — the first bullet of `## Notes for the plan writer` duplicates `relevant.md`. Both name the redundancy step and the redundancy note as templates, and `relevant.md` is where the head start belongs. Cut the bullet or keep it to one line.

8. **LOW** — `## Open risks` bullets 3 and 4 repeat D1 (map commit on every answer) and D4 (the tier 2 and spawn caveat) without adding anything new. Bullet 2 adds the minutes-per-package cost and earns its place. Fix: cut bullets 3 and 4, or merge them into one line about the staleness of the stored maps.

9. **LOW** (outside leanness): D6 and D8 share the verb name `lym gate reach` but take different argument shapes. One is positional `<file>#<Func>`; the other uses `--test` and `--since`. Neither Decision says what the verb does when it gets both, or neither. That is a bound gap, not a leanness issue.

Further tool calls: none.
