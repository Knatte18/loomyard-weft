```yaml
lens: fix-consistency
outcome: reported
```

no findings

Checked:
- The commit touches only `internal/boardengine/handles_test.go` (3 insertions, 1 deletion).
- The new `nohandle` note reuses the `{"kind":"note","labels":["removed-type"]}` shape that `TestTitleWithHandle`'s `AnEntryWithNoHandleKeepsTheName` row already uses for an entry with no handle.
- The doc comment on `TestWorkspaceFolderNames` ("an entry the board cannot give a project handle is left out") still describes the fixture, and the `want` map correctly omits `under-nohandle`.
- The `handles.go` doc comment ("whose project the board does not hold with a handle, is left out") matches the new row.
- The sibling fixtures in `TestWindowTitles` and `TestRecipeWindowTitle` exercise other functions, so their twins need no matching edit.
- `ReadTasks` assigns handles to projects but not to an untyped note, so the proposed project fixture could not reach the empty-handle branch. A note can, and the gate passed with it.

Tool calls:
1. `git show 8b64be72 --stat`: confirmed the commit's file scope, which my context did not show.
