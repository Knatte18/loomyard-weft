```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds, the test that pins it, or none.

**Behaviour to test**
- Upsert of a new task from a run agent is refused: `TestTaskCreationGuard/upsert a new task` (`taskguard_test.go`).
- Upsert turning a note into a task is refused: `upsert a note to kind task`.
- Promote is refused: `promote a note`.
- Upsert-batch carrying a new task is refused: `batch carrying a new task`.
- Merge upserting a new task is refused: `merge upserting a new task`.
- A refused write leaves `board.json` byte-identical: the shared before/after comparison in the refused rows.
- The message names the strand, the entry and `"kind":"note"`: the shared `strings.Contains` loop over the refused rows.
- A run agent's new note passes: `a run agent files a note`.
- A run agent's edit of an existing task passes: `a run agent edits an existing task`.
- A run agent's rename passes (the loaded slug is judged as renamed): `a run agent renames a task`.
- Empty strand name passes: `the operator creates a task`.
- Hub orch (`ly:orch`) passes: `the hub orch promotes a note`.
- Orch thread (`ly:some-task:orch`) passes: `an orch thread creates a task`.
- Slug-free name with a non-orch role (such as `ly:webster`) passes: no row. It is a `Slug == ""` branch of `isRunAgentSession`, and only the `ly:orch` row pins it.
- A strand name that does not parse passes: no row.
- `Board.strandName` and `Config.StrandName` plumbing in `New`: pinned indirectly, since every engine row sets `config.StrandName` and would fail if the plumbing broke.
- `cfg.StrandName = strandNameFromEnv()` in the CLI: `stepTaskSessionFilesNotesOnly` (`verbs_test.go`), registered in `cli_test.go`. It checks that `upsert` of a task under `ly:task-a:loom` exits 1 with the strand name and `"kind":"note"` in the message.
- Stencil and rubric prose, and the doc.go text: no tests, and none are needed (static prose is exempt).

**Test economy**
- No new test duplicates an existing test. `taskguard_test.go` is the only test of the new guard, and the CLI step covers a different seam (env to config).
- One table-driven test covers every path, which fits the one-test-per-behaviour rule.
- The CLI step overlaps the engine rows on the refusal itself. It is justified because the acceptance criteria ask for the CLI wiring to be shown.

**Findings**

1. NIT, `consistency`, `internal/boardcli/verbs_test.go:109`, `stepTaskSessionFilesNotesOnly`.
   - Its doc comment says the run agent's note "is stored".
   - The step only runs `runJSON(t, 0, "upsert", ...)` and never reads the note back or checks that the refused task left nothing.
   - Fix: assert the stored entry (a `get`/`find`, or the returned task's `kind` being `note`), or reword the comment to "is accepted".

2. LOW, `scope` (coverage gap), `internal/boardengine/taskguard_test.go`.
   - The `isRunAgentSession` branches `Slug == ""` with a non-orch role and an unparseable name are not exercised.
   - `ly:orch` passes only through the slug-free branch, so a regression that treats other slug-free roles as run agents would go unnoticed.
   - Fix: add a row such as `{name: "a slug-free non-orch name creates a task", strandName: "ly:webster", ...}` to the existing table; a garbage-name row is optional.

Tool calls: none. The diff, the test files and the acceptance criteria were already in context.
