```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that pins it, or "none".

**Per behaviour, test that pins it**

1. Preflight's `worktree-clean` ignores untracked files under the run-records root, at anchor `.` and at anchor `backend`. Pinned by `TestCheckResolved_RunStateUnderLymIsExempt` (`preflight_integration_test.go:~414-460`). It writes the settle, tackler, arbiter, drive-report, stop-marker, condition-stop-marker, notice, status and seed files. It first asserts the unscoped `Clean` sees them, then asserts `CheckResolved` passes.
2. A dirty path under `_lym` outside the run-records root still counts. Pinned by the same test, whose last step writes `_lym/plan/plan.md` and expects `CheckWorktreeClean`. This covers a sibling under `_lym` at both anchors. The exclude pathspec fails the test if it is too wide.
3. `exemptPathspec` and `weftCommitPathspec` scoping with `AnchorRel` set to a subpath, and with `AnchorRel` set to `.`. Pinned only through the anchor `backend` row and the `.` row of the same test. There is no direct fabricengine-level test.
4. The state side still refuses an untracked file at its root. Pinned by the existing `TestCheckResolved_Failures` row `DirtyPairedSide` (`:269`). It writes `h.PrimeRecords()` (the state-side root) and expects `CheckWorktreeClean`, so it covers the "nothing else exempt" case with the exemption active.
5. `worktreeDirty` takes a variadic `pathspec` with `--` placed before it. Pinned indirectly by item 1, where a wrong `--` placement makes git error and fails `CheckResolved`. Callers that pass no pathspec keep the old command line, and their existing tests (`TestClean_ReasonWording`, `WorktreeDirty*ForTest`) still pass. No new row targets the `--` separator for the tracked scope. That scope is not reached by the new code, since `dirtyReason` uses `scopeAll`.
6. `Clean(l, weftExempt...)` with no exempt paths keeps the old unscoped check. Pinned by `cleanreason_integration_test.go:43,59,75` and `smoke_prebootstrap_integration_test.go`, which call `Clean(loc)`.
7. The exemption is state-side only (weft); the code side still counts. Partly pinned. The existing `TestClean_ReasonWording` `CodeSideOnly` row and `DirtyCodeSide*` rows pin that code-side dirt refuses. No row writes an untracked file at `_lym/shed/...` on the code side and expects a refusal. The code side is only reachable through the gitignore or junction there, so it is hard to produce.
8. `runStateExemptRels` returns the single run-records root. Pinned only by the behaviour in item 1; there is no unit test of the function, which is fine.
9. The `doc.go` paragraph is prose and is exempt.

**Findings**

- LOW, design (test economy / coverage): `Clean`'s new `weftExempt` parameter is covered only through `preflight.CheckResolved`, not at the `fabricengine` level. `grep` found no fabricengine test passing `Clean(l, <path>)` (hits were unrelated files). Evidence: `preflight_integration_test.go:444` calls `Clean(l)` with no exempt argument, and the grep showed no fabricengine test file using the parameter. Not blocking, since the public path is pinned by the preflight test.
- NIT, consistency: `TestCheckResolved_RunStateUnderLymIsExempt` has a hand-listed set of `shedrun` helpers. The Goal asks for a test that "walks the files the stop and settle paths write". Nothing ties the list to the real writers, so a new writer under the run-records root goes untested. The whole-root exemption makes a new file safe, so this is test drift only. Evidence: `preflight_integration_test.go` `runState` slice.
- No duplicate test found: `TestCheckResolved_RunStateUnderLymIsExempt` covers a behaviour no existing `Clean` or `CheckResolved` row covered, because the existing rows only write at the state-side root.

**Tool calls (1)**
1. `grep` across `*_test.go` for `weftExempt`, `exemptPathspec` and `Clean(... , "...")`, plus the `DirtyPairedSide` rows in `preflight_integration_test.go`. My context lacked any fabricengine-level use of the new parameter and the exact `DirtyPairedSide` row.
