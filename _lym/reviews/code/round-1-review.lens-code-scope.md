```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the board entry's Goal points and checked whether each point is built.

**Decisions** (the record has a decision record alone, empty sections, and no `Added after Discussion` entries):
- **G1:** after `lym loom resume`, the run-exit audit flags the earlier Master's legitimate `webster-master.md` as a parent-write and blocks the Webster row.
- **G2 (Idea):** the audit treats every friction note path a Master of this run was given as Master's own, not only the latest prompt's.

**Per change:**
- **C1** `state.go:96-100`: `MasterNotePath` becomes the list `MasterNotePaths` (`masterNotePaths`). Traces to G2.
- **C2** `runlevel.go:810-812`: each spawn appends its prompt's note path, skipping "" and duplicates, instead of overwriting. Traces to G2. This is the fix for G1's root cause.
- **C3** `audit.go:255-284`: `CheckParent` takes `notePaths []string` and allows a write to any of them. Traces to G2.
- **C4** `runlevel.go:1093` (run-exit audit) traces to G1. `recordbatch.go:413` (record-batch parent audit) is forced by C3's signature change and also traces to G2, since that is the same parent audit.
- **C5** `doc.go:127` and the `audit.go:4-5` file header: forced by C1 and C3, which made the doc text stale.
- **C6** test call sites in `acceptbatch_test.go:39` and `auditseverity_test.go:152,155`: forced by C3's signature.
- **C7** new and changed test rows, in `audit_test.go` `TestCheckParent`, the `recordbatch_test.go:585` row, and `runlevel_test.go:1675` plus the new `TestRun_DoneOutcome` resumed row: forced, since they test G1 and G2.
- **C8** commit 107dad8c2 splits over-long comment lines. Both lines are ones this branch added in `audit.go:4` and the `runlevel_test.go:1922` comment. Forced by C5 and C7.

**Per decision:**
- **G1:** built by C2, C3 and C4 (run-exit), and tested by C7's resumed `TestRun_DoneOutcome` row.
- **G2:** built by C1, C2, C3 and C4 (both audits). It covers only states written by the new binary; see finding 1.

No change traces to nothing. No `Pattern-Departure:` trailer is needed or present.

**Findings:**

1. **LOW, scope.** Location: `internal/websterengine/state.go:100`. G2 is only partly built for a run whose `state.json` predates the change. That is exactly the in-flight case the entry reports, a weaver-bench-b2 run that is resumed.
   - The old `masterNotePath` key still holds the earlier Master's note, but no reader maps it into `MasterNotePaths`.
   - A pre-change run resumed after landing starts its list with the new prompt's path alone. Its earlier note is flagged once more, which is the G1 symptom.
   - Evidence: `state.go:100` declares only the `masterNotePaths` tag. `runlevel.go:810-812` appends to whatever was loaded. `_lym/landing/summary.md` (Design decisions) states the omission as the writer's choice and points to the accept route.
   - Fix options: read the legacy key into the list on load, or confirm the omission as accepted.

Tool calls:
- `git show --stat` and `git show 107dad8c2`: my context did not show that commit's own hunks, so I could not tell which lines it split or whether they were the branch's own.
