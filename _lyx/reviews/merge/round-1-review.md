---
verdict: APPROVED
---

No finding: the merged `internal/webstercli` code, docs and tests agree with both sides.

## Focus answers

- **recover-batch ordering.**
  `recoverbatch.go` calls `c.reedUp()` (now `func() error`) and then `RecoverSpawnOrAttach`, whose `renameToolRefusal` (`internal/websterengine/recoverbatch.go:250`) maps to `rename_tool_missing` at `recoverbatch.go:212`.
  The merge left the order as each side wrote it.
  `TestRenameToolMissingEnvelope` leaves `reedUp` nil in `newVerbsFixture`, so the nil-check skips the boot and the new signature never enters the test.
  It asserts `PrepareCalls == 0`, so it pins refusal-before-spawn, which is what its doc comment claims.
  `TestRecoverBatchCmd_FabricSyncAndReedBootWayForward` was updated to `func() error` and still covers the boot refusal.
- **cli.go and wiring.go.**
  The `reedUp` field comment, the `wireStandalone` comment and the `renameTools` field comment (`wireHub` loads, `wireStandalone` leaves empty) are all true in the merged tree.
  `wireStandalone` does not assign `renameTools`, and no remaining prose mentions a watcher.
- **Budget and tests.**
  `cmd/lyx/testdata/test-budget.yaml` has `internal/webstercli: 82`, and the merged package has 82 top-level `func Test` (counted per file).
  The parent removed `TestReedUpSeam_WatcherLifecycle`, which tested a copy of the removed closure shape.
  It overlaps no task test, and the task's `TestRenameToolMissingEnvelope` and the extended `TestWireHub_LanguageSkills` cover the rename wiring.
- **docs and PATTERN.**
  The parent's `PATTERN-daemon-wakeups.md` edits concern creel and reed.
  The task adds no unbounded wait, so nothing in the task's code falls under the changed rule.
  `docs/overview.md` keeps the task's `languageconfig` rename-tools row and the parent's reed rows, and nothing in it contradicts the merged webster code.
- **Removed watcher callers.**
  A search of `internal/webstercli`, `internal/websterengine/doc.go`, `internal/burlercli`, `docs` and `contracts` for a resize watcher, `reedEngine.Watch` or a two-argument `reedUp` finds no remaining use or description.

## Rejected

- In standalone, `reedUp` boots the reed session before `RecoverSpawnOrAttach` refuses a symbol `Rename` batch, which the comment above the boot (it claims a refusal boots no substrate) does not mention.
  The order and the comment both predate the parent's change, and the task's refusal sits inside the engine call alone, so the defect lives in one side and is not this perch's subject.
