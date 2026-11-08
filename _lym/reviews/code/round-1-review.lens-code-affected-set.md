```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. I list what each change forces outside its own lines, then the findings.

**Per change: the set found, and whether it is covered**

- **`trackedDirtyPaths` split into `parseTrackedDirtyPaths`.** The callers are `originreconcilemoves.go:128`, `checkoutstate.go:55`, `pull.go:469`, `add.go:121`, `diff.go:95` and `diff.go:99`, and `mergeguards.go:116`. The signature of `trackedDirtyPaths` is unchanged, so all are covered.
- **`List` split into `entriesFromWorktrees`.** `List` keeps its signature. The grep showed no other reference to the old `TestList` in `fabricengine`; the only other hit is the unrelated `shedrun/seed_test.go`. Covered.
- **`unseedGitExclude` split into `excludeLinesToStrip` and `stripExcludeLines`.** `WarpJunctions(l, slug, names)` is no longer iterated; the new code iterates `names` directly. I could not check that `WarpJunctions` yields exactly `names` by name (its definition was outside my reads). This is not proven, so I raise it as an open question below.
- **Removed `livestate_states_selftest_test.go` helpers.** These are `resolveStateTarget`, `newPorcelainLines`, `assertOnlyNewShape`, `assertBothNewShapes`, `gitPathIsTracked`, `assertStateEstablished`, `assertTrackedSymlinkEstablished`, `structuralStateNames` (moved), and the old `TestStates`. The grep excluding that file found no remaining reference anywhere in `internal`, `docs`, `PATTERN.md` or `pattern`. `firstWiredJunction` stays defined in `livestate_manifest_selftest_test.go` and is still used there. Covered.
- **`assertArrangePreconditions` gained a `verb` parameter.** The only caller is `livestate_matrix_test.go:279`, which passes `vc.Name`. The names `"Add"`, `"Checkout"` and `"Pull"` match `livestate_verbs_test.go:396`, `:689` and `:907`. Covered.
- **`States` is now wrapped by `withVacuityCheck`, and `State.Apply` is replaced.**
  - `h.Anchor` exists (`hubforge/hub.go:175`).
  - The sync.Map key is the anchor value only. That matches "once per anchor" but not once per hub, which is the record's intent.
  - Every cell caller of `States` now takes the check on its first Apply per anchor. Covered.
- **Renamed tests.** These are `TestCleanup_VerdictIsIndependentOfDryRunAndForce`, `TestCommitWeft_...AtNestedAnchor`, and `TestCleanHubIsClean` (which replaces `TestStates`). The grep found no reference to the old names in docs or code. Covered.
- **Five `//testtiming:keep` marks.** Each carries a reason and no other file lists the marked tests. I could not check from my reads that the `//testtiming:keep` directive grammar accepts a reason after the tag. The marks follow the form the record names.
- **Rule documents.** The diff touches no `PATTERN.md` or `pattern/` file, and the record declares no PATTERN changes. `PATTERN-test-speed` and `PATTERN-test-isolation` are the entries that apply to the new tier 0 tests. They carry `tiergate.Require(t, 0)` and `t.Parallel()`. Their file headers and doc comments follow the repository's comment style. Covered.

**Findings**

1. **MEDIUM, scope.** `withVacuityCheck` dedups per anchor value across the whole process, not per hub (`livestate_states_test.go`, the `checkedAnchors` `sync.Map` keyed by `h.Anchor`).
   - The first cell to reach a structural state at an anchor takes the check; every later hub at that anchor skips it.
   - If an `Apply` regresses to a no-op on a hub shape that only later cells use, nothing fails.
   - The same gap holds for a cell filter that runs only some cells, and for the `-run` filter and for `t.Parallel` ordering.
   - The record said "into the fixtures' own apply path", which this does, but the pre-diff `TestStates` checked every state at both anchors.
   - Evidence: `livestate_states_test.go` diff hunk `@@ -52,7 +52,7 @@` onward; the commit message of c8b62f26 states the trade-off.
   - Fix: none required if the operator accepts the trade-off; otherwise key the map on `h.Path`. I mark this MEDIUM `scope` and leave the choice to the handler.

2. **LOW, consistency.** The old `TestStates` ran both anchors `.` and `backend`, and `TestCleanHubIsClean` runs `backend` only. The `livestate_states_selftest_test.go` header no longer says the clean-hub check skips the `.` anchor, but the record asks for "cut to the rows a nested anchor needs", so this is an observation, not a defect. I raise nothing further.

**Not verified:** whether `WarpJunctions(l, slug, names)` returns exactly the entries of `names` in the old `unseedGitExclude`. If it filtered `names` (for example dropping a name with no junction), `excludeLinesToStrip` now strips lines for names the old code left alone. The record requires behavior unchanged. This lens could not settle it with the reads I made. I leave it as an open question for the handler or the correctness lens.

**Tool calls**
1. One `grep` over `internal`, `docs`, `PATTERN.md` and `pattern` for the removed and renamed symbols and the `assertArrangePreconditions` callers, plus the verb `Name:` fields. My context lacked every reference outside the diff.
2. One `grep` for `Hub.Anchor` and `firstWiredJunction`. My context lacked the `Hub` struct definition and the remaining callers of that helper.
