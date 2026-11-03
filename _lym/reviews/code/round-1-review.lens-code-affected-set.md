```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the Drift-Baseline stencil change forces outside its own lines.

**Per change, the set found**

1. **Step 2 scope list** (`off` only for an unreachable task, `full` for every reached task, `partial` only for meaningless or nondeterministic tests, plus the closing "never skip" line). Covered.
   - `internal/driftengine/doc.go:73` says the baseline prompt "has the agent record scope off when the command's projects cannot reach the task's change". That is still true, because the reachability check before the list is unchanged.
   - `internal/driftengine/doc.go:119` and `verify.go:70` describe the decision halt under scope `full` for unruled acceptances. The diff's routing of intended drift to `full` makes that path more common, and the entry asks for exactly that, so nothing is left wrong.
   - `internal/driftengine/doc.go:6`, `config.go:132` and `verbs.go:59` hold the `partial`-needs-a-skip-list rule. It is unchanged and agrees with the new `partial` wording.
   - `internal/driftcli/cli.go:93` and `template.yaml:9` carry only scope-verb examples and config key notes, with no guidance on when to choose a scope. Covered.
   - The configured `drift.scope` path writes the scope with no agent (`doc.go:84`). It never reads this stencil, so it is unaffected.
   - Repo-wide grep for "on purpose", "intended change", "scope off" and "scope partial" found no other doc, spec or stencil that repeats the old guidance of `off` or `partial` for intended change. The `docs/research/` hits are dated reports, so they are exempt.

2. **Step 3 and the `scope_note` marker.** Unchanged. `contracts/stencils/markers_test.go:62` lists the same baseline markers, and the diff adds and removes no `{{...}}` marker. Covered.

3. **Step 4 report wording** (name each drifting test, the lines or values that change and why, and say every other line stays; name each skipped test and why).
   - It agrees with `drift-template-verify.md:29-31`: "Drift the task intends is acceptable; drift it does not intend is a defect".
   - It agrees with `autonomousVerifyJudging` (`seats.go:57`), which accepts with `lym drift accept <test> --reason`.
   - It agrees with `operatorVerifyJudging` (`seats.go:59`), which presents "the baseline report's expectation for the test".
   - The verify stencil's line 23 says that when no baseline agent ran, the report "names only the scope and its skipped tests". The new Step 4 asks for the same two things, so the stencils do not contradict.
   - No contradiction found.

4. **Stencil ownership (`PATTERN-stencil-ownership`).** The change edits the `contracts/stencils` embedded default, which `stencilstore` seeds and reads at call time. No second copy of the prompt and no embedded bytes were added. The diff restates no other producer's format. Covered.

5. **Declared PATTERN changes.** The directive declares none, so there is nothing to land. `PATTERN.md` and `pattern/` are untouched, and no rule is added or removed. Covered.

6. **Commit message `e12efa51` and `_lym/landing/summary.md`.**
   - Both match the diff: the `off`, `full` and `partial` routing, the `--skip` reservation, and `baseline.md` naming the expected lines.
   - The summary's "No Go code changes" and "`drift.scope` path untouched" are true.
   - No `Pattern-Departure:` trailer is present, and none is needed.

**Findings**

- **NIT, `scope`**, `contracts/stencils/drift/drift-template-baseline.md:52`. "Name each skipped test and why the task makes it meaningless or nondeterministic" is not asked for by the entry's Expected section. When the scope was already recorded by the parent (`scope_note`, Step 4's last sentence), the skip list may have been chosen for another reason, so the "why the task makes it meaningless or nondeterministic" phrasing presumes a reason the agent cannot know. Mild and consistent with the verify stencil's line 23. Keep or drop at the handler's judgment.

No other member of the affected set is left wrong.

**Tool calls:** none. Everything came from the inherited context.
