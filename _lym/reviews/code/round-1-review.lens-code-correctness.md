```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, running each changed branch on a concrete input. I made no tool calls and found nothing wrong.

**Per function or branch, with the case run:**

- `isDraft` (`draft.go:14`):
  - Status `nil` returns false, so the task falls to Ready, as before.
  - Status `""` returns false. The write rule turns empty into `draft`, so a read of an empty status is a transient case, not a new one.
  - Status `"draft"` returns true.
  - Correct.
- `ComputeLayers` pre-assignment (`layer.go:36-47`):
  - A draft task with no dependencies gets `Draft`.
  - A draft task with an open dependency gets `Draft`.
  - An isolated draft task gets `Draft`, since the draft check comes before the isolated check.
  - A draft task a run holds stays `Running`, because `RunHolds` and done are checked first.
  - A done task whose status is `draft` stays `__done__`.
  - A draft project stays `Project`.
  - Correct.
- `ComputeLayers` depth (`layer.go:108-142`):
  - A task depending only on a draft task: `getDepth(draft)` returns 0 because `layerMap` is already set. `maxDepth` is 0, so `d` is 1, which is `A`.
  - A task depending on a Ready task and a draft task is also `A`.
  - A chain of draft tasks adds no letters, because each contributes 0.
  - The `A..Y` cap is unaffected.
  - Correct.
- `ComputeLayers` cycle detection (`layer.go:56-90`):
  - The DFS walks `taskMap`, which holds every task, so a draft task is not skipped.
  - A cycle through draft tasks (`a→b→a`, both draft) still returns the "cycle detected" error.
  - A draft task that depends on a running or done task is handled as before.
  - Correct.
- `RenderOrder` `bucketOrder` (`layer.go:201`):
  - `A..Y` map to 2..26, Z to 27, Draft to 28 and done to 29.
  - Running and Ready keep 0 and 1.
  - The sort puts done last first, then kind, then bucket, so Draft sorts after Z and before done.
  - A draft with a lower handle than a Z task still sorts after it.
  - Correct.
- `renderReadme` layer loop (`render.go:318-330`):
  - The run-grouping loop emits one `### Draft (n)` section per contiguous run, so Draft comes last among task sections.
  - With no draft task there is no run, so the section is omitted.
  - The Ready count uses `byLayer(readyLayer)`, which excludes draft.
  - Ready renders `_None._` when empty, as the updated golden shows.
  - Correct.
- `layerSection` (`render.go:468-480`):
  - `draftLayer` returns the name `Draft` and the meaning line.
  - Layer A's meaning line is updated to name Draft.
  - Correct.
- `standingCell` (`render.go:443-456`):
  - A draft member, isolated or not, falls through to `layerSection(member.Layer).name`, which is `Draft`.
  - A running draft member returns the run place, because `Running` is matched before the fallthrough.
  - A finished member returns `Done`.
  - Correct.
- `entryCell` (`render.go:520`): swapping to `isDraft` leaves behavior unchanged, and the `draft` code span stays. Correct.

No findings.

**Tool calls:** none. The manifest context already held the diff, `layer.go`, and the `render.go` regions I needed.
