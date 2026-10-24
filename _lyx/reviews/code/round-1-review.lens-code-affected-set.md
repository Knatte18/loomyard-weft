```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change, I list the set I found and whether the diff covers it.

**Per-change list**

1. **New `Config.StrandName` field, passed through `New` into `Board.strandName`.**
   - Covered: the only CLI constructor is `internal/boardcli/cli.go:115`/`:145`.
   - Covered: `battencli/wire.go:1211` and `:1238` and `battencli/owner.go:63` build a `Config` without a strand name, so the guard is off there. That matches the record's open-risk note.

2. **`checkTaskCreation` in the shared `boardCriticalSection`, which every mutating verb uses.**
   - Covered: callers are `UpsertTask`, `UpsertTasksBatch`, `Promote`, `MergeTasks`, `Rename`, `intake.go:89` and `siblings.go:105`.
   - `intake.go:197` files notes only.
   - The `Rename` row passes under a run-agent name (`taskguard_test.go`, "a run agent renames a task").
   - Not checked: the `siblings.go` path, which has no test row. I did not see what it writes.

3. **Every `lyx board` write verb now carries `LYX_STRAND_NAME`.**
   - Covered: the existing task-session tests at `verbs_test.go:669` and `:684` only edit or re-send an existing task, so the guard doesn't break them.
   - Covered: no other test file that sets `StrandNameEnv` calls the board writes.

4. **Stencils (Discussion, Plan, Darn, two rubrics).**
   - Covered: each writer's own-entry paragraph gets a matching note-never-task sentence, and each rubric gets a matching bullet.
   - Covered: the orch stencil is untouched, as the D5 withdrawal requires. Its `lyx board upsert` use at `orch-template-role.md:163` and `:205` is exempt, since the orch role is exempt.
   - Covered: no other run-agent stencil tells an agent to file a task, from my grep of `contracts/` for `board upsert|promote|merge`.

5. **Docs.**
   - Covered: `boardengine/doc.go` and `boardcli/doc.go`.
   - **Not covered:** `plugins/ly/skills/board/SKILL.md`. The agent-facing board skill documents `upsert`, `merge` and `promote` with `kind":"task"` examples (`SKILL.md:58-88`). Its "Claimed entries" section (`:120-127`) documents the analogous run-lock refusal. The new behavior is not mentioned. CLAUDE.md says a task that changes observable CLI behavior updates the docs in the same commit. This is the doc a run agent reads.

**Findings**

- **LOW, consistency (scope).** `plugins/ly/skills/board/SKILL.md:58-127`. The skill documents `lyx board upsert`, `merge` and `promote` with no note that a run agent's session is refused when a write creates or promotes a task. The skill teaches `promote` and task-kind `upsert` without that caveat.
  - Fix: add a short paragraph under "Claimed entries" or a new "Run sessions" heading. It should say a session whose `LYX_STRAND_NAME` carries a task slug under a non-orch role cannot create or promote a task, and that the follow-up goes in as a note with `"kind":"note"`.

No other member of the affected set is left wrong.

**Tool calls (4 of 6)**
1. grep of `contracts`, `plugins`, `docs`, `PATTERN.md` and `pattern` for `board upsert|promote|merge|upsert-batch`, to find other stencil or doc users. My context didn't hold them.
2. grep of tests setting `StrandNameEnv`, and non-test `boardengine.Config{` constructors, to find callers that could be affected.
3. Read `SKILL.md` lines 84-135, plus `verbs_test.go:655-705` and `cli_test.go:95-135`, to check the board skill's refusal docs and the existing task-session tests.
4. grep of `Kind` in `intake.go` and `siblings.go`, to see whether the other critical-section callers create tasks.
