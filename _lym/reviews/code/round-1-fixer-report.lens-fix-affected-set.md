```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set lens, checking that the fix covers every member of the sets behind F1, F4, F6 and F8.

**No findings.**

Unchecked:
- F3 is the parent's decision.
- F9's Windows path was not run, and I did not verify that the comment-only change matches the review's second option.
- F10's accepted gap was not searched; `measurePackage` has no test.

Sets checked:
- **F4, testmap request sites:** the only `testMapRequest{` literals are the production one in `testmap.go:113` and three test ones. All four set `cappedParallel`, and `r.sideBySide` is assigned only in `takeSlot` (`testmappackage.go:607`).
- **F4, direct `testMapRun{}` literals:** `testmap.go:255` sets `request`, and `testmaporder_test.go:30` only calls `orderJobs`, which never reaches `cappedParallel`.
- **F4, stale `r.parallel` uses:** the remaining uses are the `-p` of the `lym` build and the slot read in `takeSlot`.
- **F1, `RunMemoryCapped` callers:** the real callers are `internal/testcover/runner.go:271` and the proc tests. None sets `ExtraFiles`; the only other `ExtraFiles` hit is the new scope path itself.
- **F4/F6, `MemoryLimit` sites:** the sites are `redundancy.go:540`, `testmappackage.go:673`, `redundantmutants.go:337` and `:452`, and `cmd/testtiming/redundancy.go:331`. The two `gatecli` run sites that were not clamped before (testmap and the report step) are now covered: testmap is clamped, and the report step runs one test at a time.
- **Docs:** `docs/overview.md:944-945`, `gatecli/doc.go` and the template comment all carry the "fewer when memory is short" wording. `gateslot/doc.go:176-181` names only the report step and `lym gate redundant`; it did not mention testmap before this change either, so it is not a stale sentence the fix introduced.

Calls made:
1. Grep for `testMapRequest{`, `testMapRun{`, `cappedParallel` and `.parallel` in `internal/gatecli`, `RunMemoryCapped(` and `ExtraFiles` repo-wide, and the side-by-side and memory wording in docs. My context lacked a repo-wide view of these sets.
2. Read `gateslot/doc.go:174-182`, `docs/overview.md:936-948`, every non-test `MemoryLimit:` site, and `testmaporder_test.go:25-40`. My context lacked these spans and the cross-repo `MemoryLimit:` set.
