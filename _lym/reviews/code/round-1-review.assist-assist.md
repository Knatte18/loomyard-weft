---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/boardengine/store.go:509
    summary: Dependencies targeting notes remain forbidden despite the added decision.
    fix: "Remove the note-target and dependent-note refusals in Store.validateWrite and the corresponding depends-note refusal in internal/raddleparser/check.go. Keep existence and cycle validation. In dependency.go, keep a present note pending even when its status is done, until promotion and task completion or removal. Update the existing refusal cases in store_test.go and raddleparser/check_test.go to accept note targets, and cover cycles through notes and the waiting lifecycle. Align boardengine documentation, plugins/ly/skills/board/SKILL.md and contracts/specs/raddle-proposal-spec.md with the permitted relationships."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/boardengine/store.go:777
    summary: Removing an entry leaves its slug in surviving entries' dependencies.
    fix: "Make Store.RemoveTask strip the removed slug from every surviving entry's DependsOn in the same mutation, preserving unrelated dependencies. Check MergeTasks, which calls RemoveTask, so cleanup applies to entries actually removed without losing references to a slug retained by the merge. Extend existing removal coverage to assert cleanup for task and note dependents and successful subsequent writes."
  - id: F3
    severity: BLOCKING
    class: consistency
    origin: lens:code-tests
    location: internal/boardengine/siblings_test.go:95
    summary: Two sibling-test rows assert the same note dependency behavior.
    fix: "Remove the new set-deps on a note waiting on what an open task waits on row from TestWriteSiblings. Retain the note takes a dependency and makes no sibling row, including its no-sibling and stored-dependency assertions."
  - id: F4
    severity: BLOCKING
    class: consistency
    origin: lens:code-tests
    location: internal/boardengine/store_test.go:1983
    summary: Independent new promotion subtests omit the required parallel execution.
    fix: "Add t.Parallel at the start of the added promotion subtest and each of its table subtests in TestPromote. Add it to the changed note-acceptance subtest in TestKindDependencyRules at line 124 too; these cases use independent in-memory stores and have no process-global fixture."
---

### [BLOCKING] Permit dependencies on notes across validation paths

**Location:** internal/boardengine/store.go:509
**Issue:** The added decision permits tasks and notes to depend on notes.
The validator still rejects either relationship and refuses turning a depended-on entry into a note.
The independent raddle validator also rejects these targets at internal/raddleparser/check.go:386.
The board skill at line 258 and the raddle proposal spec at line 99 preserve that prohibition.
The existing dependency model treats any done entry as satisfied, including a note that has never become a task.
The fix must therefore cover waiting semantics as well as admission.
This consolidates the correctness, affected-set and scope lenses' reports of the same omitted policy change.

### [BLOCKING] Remove incoming dependency references with the entry

**Location:** internal/boardengine/store.go:777
**Issue:** RemoveTask deletes only the selected entry.
For a note waiting on task dep, removing dep leaves the note's DependsOn unchanged.
The new Pending field then omits dep because the dependency model treats absent entries as met,
but the README still displays it and a later validated write rejects the dangling dependency.
Board.RemoveTask delegates directly to this method, without additional cleanup.
The added decision explicitly requires cleanup for every surviving entry.

### [BLOCKING] Consolidate duplicate sibling coverage

**Location:** internal/boardengine/siblings_test.go:95
**Issue:** This row and the row at line 226 both set a note's dependency to an entry an open task waits on.
Both require no sibling notice and assert the dependency was stored.
Their different slugs and insertion order establish no distinct asserted behavior.
PATTERN-test-speed requires nonredundant behavior coverage.

### [BLOCKING] Run independent subtests in parallel

**Location:** internal/boardengine/store_test.go:1983
**Issue:** The new promotion subtest and its rows create independent in-memory stores but omit t.Parallel.
The changed note-acceptance subtest at line 124 has the same omission.
PATTERN-test-speed requires parallel execution unless a named process-global fixture prevents it.
No such fixture is used by these cases.

The original note-owned dependency behavior is implemented for writes, promotion, pruning and display.
The README and compact-output branches were inspected directly;
human-only layout and wording assertions are excluded by the standing testing rules.
The changed refusal tests now assert the requested acceptance behavior, rather than relaxing an unrelated limit.

Validation limitation: the attempted tier-0 gate could not acquire its slot because the hub gate directory is read-only.
No tests ran in that attempt.
The supplied gate reports and source inspection informed this review; no new passing test run is claimed.
