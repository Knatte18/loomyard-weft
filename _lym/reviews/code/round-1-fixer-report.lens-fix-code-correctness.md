```yaml
lens: fix-code-correctness
outcome: reported
```

Load does not call `normalizeTask`, so a task entry with `beam:true` loads as written. The gate passing confirms it.

No findings.

Per-row trace:
- `BoardNonProjectWithBeamFlagReadsTask`: the origin has no beam mark, so `pairIsBeam` falls to the board. The `child-task` entry is `task` with `beam:true`. The intact check gives `false` and the place is `ButtonPlaceTask`. With `task.Kind == boardengine.KindProject` deleted it gives `true`, the place becomes `ButtonPlaceBeam`, and the row fails.
- `BoardBeamOfAnotherSlugReadsTask`: the only entry is `other-beam`, a project with the beam flag. The loop never matches `child-task`, so it falls through to `false` and `ButtonPlaceTask`. A `pairIsBeam` that matched any beam entry returns `true` and the row fails.
- `boardWith` is a closure local to the test function. The only callers are the four board rows, and all four pass the new slug argument. Nothing else breaks.

Calls made:
1. `grep` of `.Beam` in `boardengine` non-test code. I wanted to know whether loading the board normalizes away the beam flag on a non-project entry, which would let the new kind row pass with the defect in place.
2. Read of `task.go:300-343` (`normalizeTask`). It rejects a task with `beam` set, so I needed to know whether `Load` calls it.
3. `grep` of `normalizeTask(` callers and `ReadTasks`. The callers are only the add, update and replay paths, and `ReadTasks` calls `Store.Load` then `ListTasksFull`. Load does not normalize, so the row loads as written.
