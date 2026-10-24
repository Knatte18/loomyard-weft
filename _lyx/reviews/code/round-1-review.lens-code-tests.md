```yaml
lens: code-tests
outcome: reported
```

Task: lens code-tests. For each behaviour the diff adds or changes, name the test that pins it, then apply test economy.

**Per behaviour, the test that pins it**
- **The ctrl+alt+a entry (key, command, args `"reed attach"`, no `when`):** `TestKeybindingsBlock` (`keybindings_test.go:22`). Without it, no test would fail if the entry were dropped or changed. `TestMergeKeybindings` and `TestSeedKeybindings` build their expectations from `keybindingsBlock(...)` (`:33`, `:111`), so they stay green either way.
- **The four Alt-arrow lines render unchanged (args, `when`, escape quoting):** `TestKeybindingsBlock` (`:18-21`). Before the diff, no test pinned the literal rendering, so this covers a gap. It also pins CRLF joining and the begin and end markers.
- **Block entry shape and rendering from the command, args and optional `when` fields:** `TestKeybindingsBlock`. Both the `when` and the no-`when` branches are in its expected text.
- **An older lyx block is replaced in place by the new block:** `TestMergeKeybindings`, row "an existing block is replaced in place, markers included" (`:~62`). `staleBlock` there differs from the current block, so the row does not depend on the block's content. It is already pinned and needs no new test.
- **The task still has one task, no `runOptions` and no `dependsOn`:** `config_test.go:64` (the loop over `runOptions` and `dependsOn`) and `:153` (single task, label). Both are unchanged and still apply.
- **The label constant feeds both the task and the binding:** the pinning is indirect.
  - `config_test.go:54` asserts the literal `"reed attach"` for the task label.
  - `keybindings_test.go:22` asserts the literal `"reed attach"` for the binding args.
  - If the constant were changed, both tests would fail, but only because their literals differ from it.
  - If `writeAttachTask` or the entry stopped using the constant and hard-coded another string, the matching test would fail.
  - No test compares the two outputs directly. That is acceptable, since each literal pins its side.
- **`Spawn` seeding the block (`ideengine`):** `internal/idecli/cli_test.go:68` and `spawn_hub_integration_test.go:170` check that the file exists and contains the begin marker and `sendSequence`. These are unchanged and do not cover the new entry, which `TestKeybindingsBlock` covers at the unit level. Nothing is missing.
- **Docs-only changes:** no test is needed.

**Findings (test economy)**
- **No findings at BLOCKING or MEDIUM.**
  - `TestKeybindingsBlock` does not duplicate an existing test. No existing test pins `keybindingsBlock`'s literal output.
  - It is one test with exact equality, not one per helper.
- **LOW, class `consistency`, `internal/vscode/keybindings_test.go:22`:** the test hard-codes `"reed attach"`, as `config_test.go:54` does. The two literals are independent pins of one coupling, which the record requires to hold. This is a note only. A single assertion that the entry's args equal the generated task's label would tie them together more directly, but I don't recommend a change, and this is not a defect.

**Tool calls**
1. `grep` over `config_test.go` and `keybindings_test.go` for the label, `runOptions`, `dependsOn`, `keybindingsBlock` and the entry strings. My context lacked `config_test.go`'s assertions and the test list.
