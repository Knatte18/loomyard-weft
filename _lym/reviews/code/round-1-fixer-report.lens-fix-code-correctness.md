```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I ran the F1–F7 changes step by step. One finding, and the F1, F3, F5, F6 and F7 changes held.

**F2 test passes with the defect in place (MEDIUM)**
- **Changed text:** the new row "entry already carrying the status writes nothing" in `TestSetRunStatus`, `internal/boardengine/board_test.go`. It asserts byte equality of `board.json`.
- **Problem:** the row does not fail when the `task.Status != nil && *task.Status == status` clause in `board.go:197` is removed.
- **Evidence:**
  - Without the clause, `SetRunStatus` calls `store.SetStatus(slug, &status)`.
  - That sets `s.tasks[i].Status` to the value the entry already holds, and writes no `Revised` stamp (`store.go:629-642`). The only `Revised` handling I found is at `store.go:738`.
  - The save would then rewrite a `board.json` that matches the seeded bytes, so the `unchanged` assertion holds either way.
  - The row's other assertion, that the status reads "running · Webster", also holds either way.
- **Fix:** make the skipped write observable. For example, seed the entry with a `Revised` value, or a field that a real write changes, and assert it survives. A real write may restamp `Revised` through some path I did not find, so check that before choosing.

**Other changes, no findings**
- **F1:**
  - The `Unreleased` closure returns the status only for `draft` or `discussed`, and `""` for a nil status or an absent slug.
  - Its only caller is `refuse`. Both the exact-text refusal rows and the resume row (`found` with an unreleased status) still pass.
  - The beam project row skips the check, as before.
- **F3:**
  - The failing-stub test fails if the seam returned the board error, since it expects `nil` and one commit and spawn.
  - The closure test fails if the run id did not map to the slug, since the entry's status would stay unset.
- **F4:**
  - The skipping-gate row catches a broken `started` recomputation. Without it, a spurious start write would add a `{B, running}` call.
  - The holding-gate row does not depend on the recomputation, because `markEntered` leaves `CurrentProducer` empty. It still pins that the first row is named once.
- **F5, F6, F7:** the new assertions match the behavior I read. The render check is guarded by `discussed >= 0 && draft > discussed`, and the original index checks fail first if the sections are missing.

**Tool calls (2 of 6)**
1. Grepped `Store.SetStatus` in `internal/boardengine/store.go` for the F2 argument; the fixer never showed that file.
2. Grepped `Revised` and `noWrite` in `internal/boardengine/board.go` and `store.go` to see whether a same-value write changes the bytes; neither file's relevant parts were in context.
