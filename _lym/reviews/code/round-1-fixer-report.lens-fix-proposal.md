```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's F1 and F2 proposals as carried out in `internal/hubgeom/hubgeom_test.go`.

**F1: row `BoardNonProjectWithBeamFlagReadsTask`** (`boardWith("child-task", "task", true)`)
- LOW: the row pins a board state a real board cannot hold. `normalizeTask` (`internal/boardengine/task.go:336`) refuses `beam` on a non-project kind on every write and replay path (`task.go:448`, `task.go:510`, `replay.go:550`, `replay.go:618`).
- Evidence: the gate passed, so the load path used by `ReadTasks` does not normalize and the row reaches `pairIsBeam`.
- With the kind operand deleted, the row returns beam and fails, so it does catch the defect. The cost is that it relies on a hand-written `board.json` that only a hand edit could produce.
- The reviewer's proposal is still the only way to pin that operand through the public surface, so I would keep it. A one-line comment on the row, saying it is a hand-edited board, would help a reader.

**F2: row `BoardBeamOfAnotherSlugReadsTask`** (`boardWith("other-beam", "project", true)`)
- No findings. The record carries no beam mark, so a `pairIsBeam` that matched any entry, or the first one, would return beam and fail the row.
- A `pairIsBeam` with the right slug match but a wrong fall-through would still pass, but the review did not ask for that case.
- The `boardWith` signature change updated both existing call sites, and the compiled test gate passed.

Tool calls:
1. grep of `Beam` in `internal/boardengine/store.go` and `task.go`: to see whether the loader clears or rejects a beam flag on a non-project entry.
2. Read of `task.go:300-350`: to read `normalizeTask` in full.
3. grep of `normalizeTask(` callers and `ReadTasks`: to see whether the read path normalizes.
