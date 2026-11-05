---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/boardengine/runlock.go:152
    summary: endsAsSketch passes a held task whole, so a write that also edits its title, body or other scope fields escapes the run lock when it ends as a status-less sketch note
    fix: >-
      In internal/boardengine/sketch.go make endsAsSketch take the vocabulary and judge the net change, not the end state alone.
      It returns true only when before.Kind is KindTask, now is a status-less Sketch, and now equals sketchOf(before, vocab) in every field scopeChanged compares (title, brief, body, short name, project, beam, order, issues, priority, front, on_land, slug) with only the fields sketchOf drops or rewrites allowed to differ.
      One way is to run scopeChanged(sketchOf(before, vocab), now, removedDone) and treat a false result as the pass.
      checkRunLocks has no vocabulary today, so pass the Board's vocab down through boardCriticalSection, or compare against the sketch label and the labels of before that are not type labels.
      Add the refusing rows listed in F6.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/battencli/startguard.go:99
    summary: The start guard no longer refuses a task whose status is the legacy discussed word, and a discussed task another entry depends on keeps that status and can be hand-started
    fix: >-
      The Goal asks for a Sketch note type and says nothing of retiring the discussed status, so keep the guard's old refusal for the legacy word.
      In hubStartGuard's Unreleased closure in internal/battencli/startguard.go return the status for both boardengine.StatusDraft and the legacy discussed word, exporting a name for the word from internal/boardengine/sketch.go (rename legacyStatusDiscussed to an exported constant) so the literal is declared once.
      Restore the refuse doc comment, the Unreleased field comment and releaseCommandText wording to name both statuses, and restore the deleted guard test row for a discussed task with no status file.
      The alternative is to migrate such a task on the read path as well, which needs a change in boardengine.ReadTasks and its callers.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/wiring.go:233
    summary: The sketchFinish fallback that leaves the entry draft when the board refuses the Sketch has no test
    fix: >-
      Add a row to TestWiredCommitStatus_FinishedRunEntry in internal/loomcli/commitstatus_integration_test.go with a motif task that another entry lists in depends_on.
      Drive the motif finish and assert the entry stays kind task with status draft, so the run lets go of the entry and nothing starts it.
      A fake Finish in wiring_commitstatus_test.go cannot reach the branch, so use the integration row.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/boardengine/sketch_test.go:60
    summary: Nothing pins the boundary of the run-lock exemption, namely that a held task which becomes a Sketch while its title or body also changes, or becomes a note without the sketch label, is still refused
    fix: >-
      In internal/boardengine/sketch_test.go add a table-driven row set over checkRunLocks or UpsertTask on a run-held task.
      One row makes the task a Sketch and changes its title, one makes it a Sketch and changes its body, and one makes it a note without the sketch label with status nil.
      Each wants ErrRunLocked through errors.Is.
      These rows fail on the current endsAsSketch and pass once F1 is fixed.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/wiring.go:233
    summary: Any MakeSketch error, a lock or save failure included, sends the entry to draft, though the comment and package doc promise that only a board refusal does
    fix: >-
      In sketchFinish in internal/loomcli/wiring.go fall back to boardengine.StatusDraft only when MakeSketch returns a refusal of the write (the validateWrite refusal), and return any other error so the seam's next finished transition retries it.
      Expose the refusal as a typed or sentinel error from MakeSketch in internal/boardengine/sketch.go for the caller to match with errors.As or errors.Is.
      Keep the warn line and the doc comments in step with that rule.
  - id: F5
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/boardengine/sketch_test.go:112
    summary: No test covers a migration of a discussed task that another entry depends on, a replayed Sketch write on a held entry, or MakeSketch on an entry that is already a Sketch
    fix: >-
      In internal/boardengine/sketch_test.go add a second task to TestDiscussedTaskBecomesASketchOnTheNextWrite that another entry depends on and assert it keeps status discussed and stays a task.
      Add a one-line already-a-Sketch row to AbsentAndDoneEntriesStay in TestMakeSketch.
      Add a replay test in the replay tests of internal/boardengine that replays a commit turning a held task into a Sketch and asserts the replay is not refused, since doc.go and the change description both claim it.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/sketch_test.go:30
    summary: HeldTaskBecomesASketch does not assert that isolated, profile, pull_request and notify_on_land are cleared, nor that project, brief, body, issues and dates are kept
    fix: >-
      In the HeldTaskBecomesASketch subtest of TestMakeSketch in internal/boardengine/sketch_test.go set isolated, profile, pull_request, notify_on_land, project, brief, body and issues on the held task.
      Assert the first four are zero after MakeSketch and the rest unchanged.
      Extend the existing subtest and add no new one.
---

### [MEDIUM] The run-lock exemption passes the whole change

**Location:** internal/boardengine/runlock.go:152
**Issue:** `endsAsSketch` returns true for any held task whose end state is a status-less note carrying the `sketch` label, and `checkRunLocks` then skips `scopeChanged` for it.
A hand upsert with a new title or body, `kind: note`, `labels: [sketch]` and `status: null` on a run-held entry therefore passes, though the Goal lets only the Sketch change through and the run lock exists to refuse other scope edits.
The change description records the exemption as a design choice, but its argument, that a hand set-status passes too, covers a status-only change and not a change to title or body.

### [MEDIUM] The start guard drops the discussed refusal

**Location:** internal/battencli/startguard.go:99
**Issue:** The diff deletes the guard's branch for the legacy `discussed` status.
A task that an older binary left discussed and that another entry depends on keeps the status after `migrateDiscussedStatus`, and `lym batten run` on it now passes the guard, as does a run on any discussed task before the next board write migrates it.
The Goal asks for a Sketch note type and the handoff of a motif run's entry to it; it does not ask to retire the status or to loosen the guard, so the change is not licensed beyond the migration itself.

### [MEDIUM] The draft fallback of a refused Sketch has no test

**Location:** internal/loomcli/wiring.go:233
**Issue:** The Goal wants the entry left a draft when the board refuses the Sketch, and the change description admits the branch is covered only by reading the code.
No integration row has an entry that another entry depends on, and a fake `Finish` cannot reach the branch.

### [MEDIUM] The exemption's boundary is unpinned

**Location:** internal/boardengine/sketch_test.go:60
**Issue:** Only the clean `MakeSketch` change is tested through `checkRunLocks`.
No row tries a held task that becomes a Sketch with a title or body edit, or a note without the sketch label, so the over-wide pass of F1 goes unseen.

### [LOW] The draft fallback runs on every MakeSketch error

**Location:** internal/loomcli/wiring.go:233
**Issue:** `sketchFinish` treats every `MakeSketch` error as a refusal.
After a post-save render failure the entry is already a Sketch, and `SetRunStatus` on a note then fails too, so the entry is left unreported by the fallback.
The comment and the loomcli package doc present the fallback as the answer to a board refusal only.

### [LOW] Three behaviours with no coverage

**Location:** internal/boardengine/sketch_test.go:112
**Issue:** The migration's refusal branch for a depended-on discussed task, the replay of a Sketch write that doc.go and the change description say passes the run lock, and `MakeSketch` on an entry already a Sketch have no test row.

### [LOW] The Sketch conversion's cleared and kept fields are under-asserted

**Location:** internal/boardengine/sketch_test.go:30
**Issue:** The held-task subtest asserts kind, labels, handle, status, recipe, `depends_on`, `run_settings`, the discussed flag and the title, but not the other cleared fields or the kept ones.
A regression that kept `isolated` or dropped `Project` would pass.

## Rejected

- Hub board.yaml with a type starting with `S` fails to load (code-correctness F2): the Goal fixes handle letter `S` for `sketch`, the existing letter check names the label to rename, and the change description flags it for the operator, so the refusal is the design and not a defect.
- A Sketch can be started by a hand `lym batten run` (focus item on startguard): no defect shown, since a Sketch is a note with no status, `sketchOf` clears the fields a run reads, and the diff leaves the start path of a note as it was; what the diff does change is covered in the start guard finding above.
- Stale documentation or code references to the removed status, layer and README section (focus items): the lens of the affected set and my own search of code, specs, skills and docs found none left.
- Test economy: no new test duplicates an existing one; `stepListShowsASketch` overlaps the `RenderCompact` row but alone reaches the hub CLI with no board.yaml type list.
