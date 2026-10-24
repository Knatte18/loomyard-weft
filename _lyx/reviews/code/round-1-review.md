---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/boardengine/taskguard.go:14
    summary: isRunAgentSession compares the role to the orch role exactly, so a numbered second orch thread such as orch-2 is refused creating a task
    fix: In isRunAgentSession replace name.Role != agentname.RoleOrch with !agentname.MatchesRole(name.Role, agentname.RoleOrch), the matching internal/fabriccli/roleguard.go already uses for the orch role. Add a ly:some-task:orch-2 row that passes to TestTaskCreationGuard in internal/boardengine/taskguard_test.go.
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: scope
    location: internal/boardengine/taskguard_test.go:TestTaskCreationGuard
    summary: The slug-free non-orch branch of isRunAgentSession has no row, so only ly:orch pins the exemption and a regression that refuses other slug-free roles would pass
    fix: Add a row to the TestTaskCreationGuard table with strandName ly:webster that creates a task and passes. The row for the F1 fix and this row are both cheap additions to the same table.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: consistency
    location: plugins/ly/skills/board/SKILL.md:120
    summary: The agent-facing board skill documents upsert, merge and promote with task examples and a run-lock refusal, but not the new refusal of a task created or promoted from a run agent's session
    fix: Add a short paragraph beside the "Claimed entries" section of plugins/ly/skills/board/SKILL.md. It says a session whose LYX_STRAND_NAME carries a task slug under a role other than orch cannot create or promote a task, and that the follow-up is filed with "kind":"note".
    origin: lens:code-affected-set
  - id: F4
    severity: NIT
    class: consistency
    location: internal/boardcli/verbs_test.go:109
    summary: The doc comment of stepTaskSessionFilesNotesOnly says the note is stored, but the step never reads it back
    fix: Reword the doc comment to say the note is accepted, or read the entry back and assert its kind is note.
    origin: lens:code-tests
---

### [MEDIUM] Numbered orch threads are treated as run agents

**Location:** internal/boardengine/taskguard.go:14
**Issue:** `isRunAgentSession` exempts the orch role by `name.Role != agentname.RoleOrch`. A second orch thread in a task worktree is named with a numbered role (`orch-2`) by `NumberRole`, so it passes the slug test and the role test and is refused. The decision record's D2 and its acceptance criteria say an orch thread in a task worktree writes as before. `agentname.MatchesRole` exists for this and `fabriccli`'s role guard already uses it. I confirmed `MatchesRole` accepts the base role and `base-N` for N of 2 or more.

### [LOW] No test row for a slug-free non-orch name

**Location:** internal/boardengine/taskguard_test.go:TestTaskCreationGuard
**Issue:** The record names an empty or slug-free name as exempt. Only `ly:orch` exercises the `Slug == ""` branch, so the branch's behavior for other roles is unpinned.

### [LOW] Board skill omits the new refusal

**Location:** plugins/ly/skills/board/SKILL.md:120
**Issue:** The skill is the document a run agent reads for `lyx board`, and it already describes the analogous run-lock refusal. CLAUDE.md asks that observable CLI behavior changes update the docs in the same commit, and the diff updates the package docs but not this one.

### [NIT] Test comment overstates the assertion

**Location:** internal/boardcli/verbs_test.go:109
**Issue:** The step asserts the exit code of the note upsert but not that the note is stored.

## Rejected

- Fail-open guard on an unparseable strand name (lens code-correctness): the run_settings and hub-config guards parse a strand name the same way, and the record bounds the guard to a mistaken write.
- Other `boardengine.New` constructions in `battencli` carry no strand name (lens code-affected-set): the decision record's open risks name this and no producer creates a task today.
- Orch stencil unchanged: correct under the withdrawn D5, not a defect.
- Commit attribution names Opus 5.5 (lens code-pattern-fit): not a rule of the target repository.
