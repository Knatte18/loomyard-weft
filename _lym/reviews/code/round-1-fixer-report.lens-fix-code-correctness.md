```yaml
lens: fix-code-correctness
outcome: reported
```

Findings:

1. **MEDIUM** — `internal/boardengine/task.go`, `ApplyPatch` (the `validateSlugLength(result.Slug)` call site, which has no reserved-slug check next to it).
   - Summary: a patch that sets `slug` to `root` still renames an existing project to `root`, so the collision with `project-root.md` stays reachable and `Render` stays silent.
   - Evidence:
     - `ApplyPatch` merges `fields` over the existing entry, so a `slug` key replaces the slug.
     - The test "patching slug beyond MaxSlugLength is rejected" shows that patching `slug` is a supported path.
     - After the merge, `ApplyPatch` runs only the empty check, `validateSlugLength` and `normalizeTask`, and none of them rejects `root`.
   - Walk-through:
     1. Take an existing project `p` and call `ApplyPatch(p, {"slug": "root"})`.
     2. No step refuses it, and it returns a project with slug `root`.
     3. `Render` then writes `projectFileName("root")` twice, and the loop's write wins over the root file.
   - The fix covered `NewTask` and `Rename` but not this third way to set a slug.
   - Adding the check here would refuse any patch of an entry that already holds slug `root`, such as a legacy entry. Limiting it to patches that change the slug avoids that.

2. **LOW** — `internal/boardengine/task_test.go`, the "slug root is rejected as reserved" subtest.
   - Summary: the subtest is correct but covers only `NewTask`.
   - Evidence: it fails with the defect in place, because without the check `NewTask` returns a project with no error. The `TargetIsRootSubject` row in `rename_test.go` likewise fails without the check. The `ApplyPatch` route (finding 1) has no row, so once it is fixed it needs one beside "patching slug beyond MaxSlugLength is rejected".

Checks that held:
- `validateSlugNotReserved` returns an error exactly when the slug equals `RootSubject`, and both call sites propagate it.
- The `NewTask` check runs after `validateSlugLength` and before the unmarshal. The error wording "reserved for the root project" matches both test assertions.
- `rootProjectFile` still evaluates to `project-root.md`, because `RootSubject` is the constant `"root"`.
- A grep for slug `root` in Go source under `internal` found no existing caller of `NewTask` or `Rename` that uses it, so no caller breaks.

Tool calls:
1. `grep` for slug `root` literals under `internal`: my context did not show whether any caller or test in other packages creates or renames an entry to `root`.
