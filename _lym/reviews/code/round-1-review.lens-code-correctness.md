```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over the changed production code and the changed test helpers. Nothing below reaches a finding.

**Per-function list**

- `parseTrackedDirtyPaths` (`dirtiness.go`)
  - **Case run:** empty stdout, a trailing NUL, and a rename record `"R  new\0old\0"`.
  - **Result:** correct. The loop body is moved verbatim, and the only change is the return type, with the error path left in the caller. An empty string yields one empty record that the existing length guard skips, so the result is nil. The rename source is consumed by `i++`.
- `entriesFromWorktrees` (`worktreelist.go`)
  - **Case run:** a nil or empty worktree slice, and an empty `Branch`.
  - **Result:** correct. The body is moved verbatim. An empty `Branch` becomes `detachedBranch`, and `List` still returns `nil` as its error.
- `excludeLinesToStrip` (`junction.go`)
  - **Case run:** names `["_lym"]`, with `sameAnchor` and `anyAnchor` each empty or set.
  - **Result:** correct. It now iterates `names` where the original iterated `WarpJunctions(l, slug, names)`. `WarpJunctions` returns one record per name with `Name: name` and no filtering, so the sets are identical. The `len(stripSet) == 0` early return in the caller is unchanged.
- `stripExcludeLines` (`junction.go`)
  - **Case run:** content with no trailing newline, and content with a blank line.
  - **Result:** correct. The logic is moved verbatim, and the trimmed-match and join behaviour is unchanged.
- `assertArrangePreconditions(t, verb, fixture)` (`livestate_matrix_test.go`)
  - **Case run:** `Add`, `Checkout` and `Pull` each with an empty field, `Checkout` with a branch equal to the original, and the `"Add/…"` and `"Checkout/…"` hostile variants.
  - **Result:** correct.
    - An empty field now stops the cell with `Fatalf`.
    - The variants (`livestate_verbs_test.go:1097`, `:1192`) match no `case`, so they skip the check, as they did before when their fields were empty.
    - A variant that did set `CheckoutBranch` equal to the original would no longer be caught, but I saw no evidence that any does.
  - **Call site:** `runCell` passes `vc.Name`, which matches.
- `withVacuityCheck` and the `States` initializer (`livestate_states_test.go`)
  - **Case run:** a first and second Apply of a structural state at one anchor, a no-op Apply, and a non-structural state.
  - **Result:** correct.
    - `checkedAnchors` is per state, captured in the closure, and keyed by `h.Anchor`, a real `hubforge.Hub` field (`hub.go:175`).
    - A no-op first Apply hits `tb.Fatalf` with the state name and anchor.
    - Non-structural states are copied unwrapped.
    - `structuralStateNames` is a package-level var read from a function called in the `States` initializer. Go's initialization-order analysis follows that reference, so the map is ready first.
  - **Race:** two parallel cells at the same anchor can race on `LoadOrStore`, but only the winner checks. Each cell has its own hub copy, so there is no shared state.
- `TestCleanup_VerdictIsIndependentOfDryRunAndForce` (`reconcile_stale_registration_test.go`)
  - **Case run:** the merged flow, step by step.
  - **Result:** correct.
    - The orphan and `unmanaged` branches are created first.
    - Both dry runs run before any mutation.
    - The first apply deletes the orphan, and the branch is re-created before the force apply.
    - `applied, err =` reuses variables already declared.
    - `weftRepoRoot` and `orphan` are in scope.
- `TestCleanHubIsClean`, the `TestList_RejectsBareRepository` rewrite and the `weftgit_exclude` single-anchor rewrite
  - **Case run:** read through once.
  - **Result:** no logic fault.

**Findings:** none.

**Tool calls**
1. `grep -A25 "^func WarpJunctions" internal/fabricengine/*.go` to see whether `WarpJunctions` filters or renames names, since `excludeLinesToStrip` now iterates `names` directly. The context did not hold its body.
2. `grep` for the `Name: "Add|Checkout|Pull"` verb cases and `Anchor ` in `internal/hubforge` to see whether the `vc.Name` switch skips the hostile variants and whether `Hub.Anchor` exists. The context did not hold either.
