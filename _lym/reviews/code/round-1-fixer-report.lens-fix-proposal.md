```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1, F2 and F3 as the fixer carried them out.

**Findings**

1. **LOW**: `NullStatusOnTask` row in `TestSetStatusReleasingProject_OtherWritesReleaseNoMember` (`internal/boardengine/projectrelease_test.go`, F2). The row doesn't pin the `current.Kind != KindProject` half of the guard at `projectrelease.go:36`, which F2 asked it to pin.
   - Remove `|| current.Kind != KindProject` and run `SetStatusReleasingProject("d1", nil)` on the seeded board.
   - `d1` is a task with status "draft". `draftMembers(tasks, "d1")` is empty because no entry has `Project == "d1"`.
   - `IsRunStatus` and `isDone` are false, and `IsEmptyStatus(draft)` is false. Control therefore reaches `return nil, s.SetStatus(selector, nil)`, the same call the guard makes.
   - The result is identical: `d1` becomes none, `d2` stays draft, `released` is nil, and the row passes with the guard half removed.
   - The row only pins a task's null write, which the later branches already cover. A task whose slug other entries name as their `project` would distinguish the two paths, but that is probably an invalid board.
   - The `status != nil` half is pinned by `NonNullStatusOnProject`: without it, `p` would be released and end as "", not "paused".
   - The change to make is either dropping the claim that the row pins the Kind guard, or seeding a task that has a draft entry with `project` set to its slug, if validation allows it.

2. **No finding** for F1 and F3.
   - F1: `stepProjectRelease` now sits above `stepProjects`, and the `stepProjects` comment block ends directly above its `func` line.
   - F3: the `DoneProjectWithoutDraftMember` row reaches the no-member message. The `RunStatusProjectNamesEveryMember` row asserts the exact `"; "`-joined string. Members are now per-row, and the refusal loop checks every member stays draft.

**Tool calls:** none. My context already held every changed file.
