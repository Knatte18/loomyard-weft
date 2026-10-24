```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds, find the test row that pins it, and apply the test-economy rules.

Per behaviour, the test that pins it:
- **Upsert of a new task refused:** `TestTaskCreationGuard`, row "upsert a new task" (`internal/boardengine/taskguard_test.go`).
- **Upsert of a note to kind task refused:** row "upsert a note to kind task".
- **Promote refused:** row "promote a note".
- **upsert-batch carrying a new task refused:** row "batch carrying a new task". The batch also holds an existing-task edit, so the row pins that the whole batch is refused.
- **Merge upserting a new task refused:** row "merge upserting a new task".
- **New note passes:** row "a run agent files a note".
- **Edit of an existing task passes:** row "a run agent edits an existing task".
- **Rename passes:** row "a run agent renames a task". It fails if the `renameSlug` handling of `loaded` is dropped, because the renamed slug would then read as a new task.
- **Operator (empty strand name) may create a task:** row "the operator creates a task".
- **Hub orch (`ly:orch`) exempt:** row "the hub orch promotes a note".
- **Orch thread exempt:** row "an orch thread creates a task".
- **Numbered orch role exempt:** row "a numbered orch thread creates a task".
- **Slug-free non-orch name passes:** row "a slug-free non-orch session creates a task".
- **Refusal message names the strand, the slug and `"kind":"note"`:** the assertion loop in `TestTaskCreationGuard`. It runs on every refused row.
- **`board.json` unchanged on refusal:** the byte comparison in `TestTaskCreationGuard`, on every refused row.
- **boardcli passes `LYX_STRAND_NAME` through to the board:** `stepTaskSessionFilesNotesOnly` (`internal/boardcli/verbs_test.go`), registered in `internal/boardcli/cli_test.go`. It fails if `cfg.StrandName = strandNameFromEnv()` is removed. It also checks that the same slug passes as a note.
- **Stencil prose, the `doc.go` text and the `SKILL.md` text:** no test, which is exempt.

Test economy:
- No new test duplicates an existing one. `stepTaskSessionFilesNotesOnly` covers the CLI wiring, and `TestTaskCreationGuard` covers the engine rule. Neither can replace the other.
- The `TestTaskCreationGuard` rows are distinct cases of one table-driven test.
- No per-helper test exists for `isRunAgentSession` or `checkTaskCreation`. Both are reached only through the `Board` facade.
- The tests of the parent's changes (`stepProfiles`, `stepRunSettings`) are outside this diff's behaviour.

Findings (LOW, below):
- **F-tests-1** — LOW, class `scope`, `internal/boardengine/taskguard.go:14` (`isRunAgentSession`), evidence `internal/boardengine/taskguard_test.go:26-110`. The `err == nil` branch has no row: a malformed or non-agent `LYX_STRAND_NAME` (for example `"garbage"`, which has one segment) passes as not a run agent. Add a row such as `{name: "an unparseable strand name creates a task", strandName: "garbage", ...}` that expects a pass. It stays LOW because the behaviour is a deliberate bound, and the doc states that the guard trusts the name.

Tool calls: none; the inherited context already held the diff and the tests.
