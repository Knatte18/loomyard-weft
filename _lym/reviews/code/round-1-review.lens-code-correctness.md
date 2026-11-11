```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. List, per function or branch, with the case run:**

1. `shedrun.RunIDs` is correct, with the shed directory absent.
   - `os.ReadDir` returns an `fs.ErrNotExist` error, so `RunIDs` returns `nil, nil`.
   - This matches the removed inline reader. `errors.Is(err, fs.ErrNotExist)` is the same test as `os.ErrNotExist`.
2. `RunIDs` is correct for a non-directory entry. It is skipped through `entry.IsDir()`, as before.
3. `RunIDs` is correct for an unreadable directory or any other error. The error is returned unchanged, as before.
4. `RunIDs` lists the same directory the old code did. The old code used `filepath.Dir(RunDir(prime, "-"))`.
   - `ResolveRunID` returns `-` unchanged, and `runSegment` returns it as one segment.
   - So the old path was `anchor/_lym/shed`, which is the path `RunIDs` now joins directly.
   - Listing the shed directory returns every name, so the legacy `self` directory appears as the name `self`.
   - A slug that is not a legacy name is not shadowed. Correct.
5. `fixloop.go` after the refactor is correct.
   - `errors`, `os` and `filepath` are still used elsewhere in the file (lines 9, 11, 12, 264 and 349), so there is no unused import.
   - The shape is the same: `RunRecordSlugs` returns `([]string, error)`. The `gofmt` re-alignment of the struct literal is fine.
6. `HeldSlugs` in `deps.go` is correct when both sources succeed.
   - `shedrun.RunIDs(prime)` and `fabricengine.TaskBranchSlugs(prime, "")` are called in sequence.
   - Prefix `""` makes `strings.HasPrefix` true for every branch, so all task branch slugs come back.
   - `TaskBranchSlugs` covers the local and the origin remote-tracking branches, as the `land.go` comment says.
   - `append(runs, branches...)` is safe: `runs` is a fresh slice, and `nil` is fine too.
7. `HeldSlugs` on an error from either call is correct. It returns `nil, err` immediately.
8. `mergeParent` on a `HeldSlugs` error is correct. It returns a wrapped error with the way forward `lym beam merge-parent <slug>` and files no task.
   - The call runs after `CodeReviewCaps` and before `FileTask`, so no state has been written yet.
9. `freeMergeTaskSlug` with an empty `held` and an empty `tasks` returns `b-merge-in`. Correct.
10. `freeMergeTaskSlug` with `held = [b-merge-in, b-merge-in-2]` returns `b-merge-in-3`. Correct, and the test pins it.
11. `freeMergeTaskSlug` with a long beam slug cuts the head to `MaxSlugLength - len(tail)`.
    - Both `taken` and the candidate use the same cut, so a held slug that is already cut matches.
    - Termination: `number` grows without bound while `taken` is finite, so some candidate is free.
    - Edge case: if `len(tail) > MaxSlugLength`, `room` goes negative and `head[:room]` panics. That needs a tail longer than the slug limit, which is not reachable in practice.
12. `taken` matches by exact slug. Held names are bare slugs: `RunIDs` gives directory names, and `TaskBranchSlugs` strips the branch prefix with `slugOfBranch`.

**Findings:** none.

Tool calls (1 of 6):
1. `grep -rn "func runSegment" -B8 -A25 internal/shedrun/`. I read it because my context lacked the definition, and I needed it to confirm that `RunDir` and `runSegment` resolve to the same shed directory `RunIDs` lists.
