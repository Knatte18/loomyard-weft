```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: checked the set-premised findings (F1, F4, F5, F11) for members the fix left out.

Checked and covered:
- **F1:** the writer of the listed file is `ReachTurn.findings` alone. The other two hits (`shedrecipe/entries_gate.go:157`, `weaverengine/strand.go:148`) only pass the path in, so the union write reaches both gates.

Findings:

1. **LOW — F5, `internal/gatecli/reach.go`, `refreshMaps`.**
   - **Summary:** the fix gave a reason only to the failed-HEAD-read branch, but two sibling branches still leave a package unrefreshed with no reason.
   - **Evidence:** in `refreshMaps`, the `staleLinked` error (`report.Errors = append(..."read the map of %s"...); continue`) and the `dirtyReason` error (`"check the files of %s"`) both skip the package without calling `unrefreshed(pkg, ...)`.
   - **Effect:** the note and `freshMaps` find no reason for those packages. The finding's premise was "the note names no reason for any unrefreshed package".

2. **LOW — F4, `internal/gatefixer/rowgate.go:116` and `:96-97`.**
   - **Summary:** two other paths run the gate over a fixer's finished tree through `waitOnly` with no re-run mark.
   - **Evidence:** `wrapped.Final = w.waitOnly(finalInner, true)` and the other entries' `waitOnly(candidate.Gate/Forked, false)` call `waitThenRun(store, live, inner, ...)` with plain `inner`.
   - **Effect:** unlike lines 157 and 210, these judge after a fixer without `w.rerun.Run`. Whether they should be marked is a judgement call: the other entries are not the fixer entry, and `Final` may not feed `WithReachTurn`.

3. **LOW — F11, `internal/testcover/coveragemap.go:176`.**
   - **Summary:** `ReadMap` still goes through `state.ReadJSON` with a read lock, so a reader still creates a `map.lock` beside a map whose lock file is absent.
   - **Evidence:** `readIndex` was changed to a plain read, but `ReadMap` is the same defect for the per-package maps. `ReadPackageMap` and `readMapDir` call it.
   - **Effect:** the `TestGateReach` unchanged-directory check passes only because `WriteMap` created those lock files first.

Calls made:
- One `grep` over `internal/testreach` and `internal/testcover`, for F11: other lock-taking readers in the maps path, which my context lacked.
- One `grep` over `internal/gatefixer`, for F4: every `waitThenRun`, `waitOnly` and `rerun.Run` caller, which my context lacked.
- One `grep` over `internal`, for F1: other writers or consumers of `ListedPath` and `reachListed`, which my context lacked.
