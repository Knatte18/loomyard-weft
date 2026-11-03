```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's changed code (F1–F5) for wrong results, broken callers, and added tests that would pass with the defect in place.

Findings:

1. **LOW** — `createRefusal(..., fabricengine.PairWorktreeStands(location, slug))` at `internal/battencli/wire.go:986` is the only place the new flag is computed.
   - The `TaskPathTakenByARegisteredWorktree` row calls `createRefusal` directly with a hand-set `true`.
   - If the caller were changed to pass `false`, or the wrong slug or location, every test would still pass.
   - The row does fail with the defect in place: the old code would return a `PreconditionError`, not the same error.
   - It only pins the function, not the wiring. Adding a test for the wiring would be a gap beyond what F4's fix asked for, so this is only a note.

2. **LOW** — The new "unlistable repository reads true" branch of `PairWorktreeStands` (`remove.go`, F3) has no test.
   - `TestPairWorktreeStands_ReadsOnlyARegisteredWorktree` still passes for registered, plain and absent paths.
   - Reverting the `List` error branch to `false` would stay green.
   - F3 is a design finding, not a coverage gap, so the fixer rule does not require a test. I list it because this lens asks whether the changed behaviour is pinned.

Traced with concrete inputs; no defects found:

- **F1:** The test for the failed refill delete:
  - Git's clean removal leaves nothing on disk, so the leftover seam is not reached on the first pass.
  - The teardown hook refills the directory.
  - In `removePair`, `pair.taskWorktree` is true, the path is present and not registered.
  - `removeWorktreeLeftover` returns the `fs.ErrPermission` error, so `refilledKept` is set and `StrayPath` equals `target`, which equals `dir`.
  - No `KindPathRemoved` entry is appended.
  - With `refilledKept` or `StrayPath` broken, the `StrayPath` assertion fails.
  - The no-op sleep seam makes it deterministic if the error were treated as a late-writer error.
- **Refactor:** `refillWithLockFile` sets `refilled` exactly as the old inline hook did, so the existing refill test is unchanged.
- **F2:**
  - A live pair at `hub/live-pair` would be reported with `CodeWorktree` equal to that path if the `liveWarpSlugs` skip broke.
  - `.hidden` is caught only by the dot-prefix skip, because `validateWorktreeSlug` accepts it.
  - `notes-weft` would be reported by the third pass with `CodeWorktree` `hub/notes-weft`, while the second pass reports `hub/notes`, so the two are told apart.
  - `_portals` is caught only by the reserved-name check.
  - Each parked path therefore fails the test if its skip is removed.
- **F3/F4 callers:** The only production caller of `PairWorktreeStands` besides `battencli` is `boardcli`'s `PairExists`, and it keeps refusing on a list failure, which is the intended direction. The only caller of `createRefusal` is `wire.go:986`, plus the test call site, which is updated. The signature change compiles, and the gate was green.
- **F5:**
  - With a plain `os.Stat` check, `strayed/obj` makes `PairExists` true.
  - The project change and rename would then be refused, so `runJSON(t, 0, ...)` would fail.
  - The test therefore detects a revert.

Further tool calls: none. My context already held every changed file.
