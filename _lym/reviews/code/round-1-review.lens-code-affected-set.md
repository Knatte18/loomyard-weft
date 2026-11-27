```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, find what calls, reads, asserts or describes it, and flag anything the diff leaves wrong.

**List, per change:**

1. **`CheckParent` signature: `notePath string` becomes `notePaths []string`.**
   - Production callers: `recordbatch.go:413` and `runlevel.go:1093`. Both are updated.
   - Test callers: `acceptbatch_test.go:39`, `auditseverity_test.go:152,155` and `audit_test.go:468,495`. All are updated.
   - Other callers: none. `webstercli` accept-audit does not call `CheckParent`.
   - Covered.

2. **What `CheckParent` allows: a write to any listed note.** Text that describes the allowance:
   - `audit.go:4-5`, the file header: updated.
   - `audit.go:256-259`, the doc comment: updated.
   - `doc.go:127`: updated.
   - `audit.go:56-57`, the `ClassParentWrite` comment: not updated. It still says "its friction note" and "the friction note its prompt names" (finding 1).
   - `audit.go:291`, the violation `Detail` string: not updated. It still says "and its friction note" (finding 2).
   - `contracts/stencils/webster/webster-template-master.md:308,319,321`: still accurate. Each prompt names one note, and these lines speak to that prompt's Master.
   - `contracts/specs`: nothing describes the allowance.
   - `runlevel.go:722`: still accurate. It is about Merriam's own prompt-time writes, not about how many notes the audit allows across a run.

3. **`State.MasterNotePath` / `masterNotePath` becomes `State.MasterNotePaths` / `masterNotePaths`.**
   - Readers and writers: `runlevel.go:810-811`, `runlevel.go:1093` and `recordbatch.go:413`. All are updated.
   - Tests: `runlevel_test.go:1675-1676`, `runlevel_test.go:2050,2064` and `recordbatch_test.go:590`. All are updated.
   - JSON fixtures, testdata and docs carrying the old key: none found outside the Go files.
   - Field doc `state.go:96-99`: updated, but line 99 still says "Empty when friction notes are off". That is no longer true once an earlier spawn recorded a note (finding 3).
   - PATTERN-tolerant-records: no refusal is added. `internal/state` keeps the unknown old `masterNotePath` key on rewrite, so there is no violation. The cross-binary effect of the rename belongs to the rubric's Merge-danger check, which the handler owns.

4. **Run appends the prompt's note path at spawn (`runlevel.go:810-812`).** Nothing else reads or writes the field. The `--fresh` and new-state branches build a new `State` (`runlevel.go:544,596`), so the list starts empty. Accept-audit loads and saves `State` whole (`acceptaudit.go:81,101,141`), so the list survives. Covered.

5. **Commits and departures.** There are no `Pattern-Departure:` trailers. `Root-Cause:` is present on 39d30e171. The directive declares no PATTERN changes, so there is nothing to land.

**Findings:**

1. LOW · consistency · `internal/websterengine/audit.go:56-57`. The `ClassParentWrite` comment still describes the allowance as one note, "the friction note its prompt names". After this diff, `CheckParent` allows every note a Master prompt of the run named. Evidence: `audit.go:56-57` against `audit.go:256-259,283`.
2. LOW · consistency · `internal/websterengine/audit.go:291`. The violation `Detail` string still says Master may write "its friction note". An operator reading a finding on a resumed run is not told that the earlier Masters' notes are allowed too. Evidence: `audit.go:291`.
3. LOW · consistency · `internal/websterengine/state.go:99`. "Empty when friction notes are off" is now wrong in one case. If an earlier spawn recorded a path and a later spawn runs with notes off, `masterNotePath` is "", so the append is skipped but the list keeps the earlier paths. Evidence: `state.go:99` against `runlevel.go:810`.

Tool calls:
- One grep of the repository, outside the Go files, for the old and new key names (fixtures, JSON, docs) and for friction-note wording in specs, stencils and docs. The manifest's grep covered only `.go` and `.md` files and did not check the specs or docs wording.
