```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I traced every function and branch the diff adds or changes with concrete inputs and found no correctness defect. One NIT and one LOW are below.

**Functions and branches traced:**

- **`beamParentTips` (`deps.go`)**
  - Case: parent has no upstream. `ReadParentTips` returns `{Local: sha, Upstream: ""}` (`parenttips.go:62-63`), there is no fetch, and `present = [local]`. Correct.
  - Case: upstream set and push skip unset. It reads again with fetch, and a fetch failure is returned unwrapped to the caller. Correct.
  - Case: local equals upstream. `present` holds the same SHA twice. That is redundant but harmless, and it matches the pre-diff loop. Correct.
  - Case: `HeadSHA` fails. The error is returned and no tips are used. Correct.
- **`ParentUnmerged` refactor**
  - Reads run in the same order as before: open, tips, refetch on the same condition, head. Error returns are unchanged.
  - Case: tip is an ancestor of head, so the result is `false`.
  - Case: `IsAncestor` errors, so it returns `false, err`. Correct.
- **`ParentConflicts`**
  - Case: an empty tips list cannot happen, because `RefSHA` on a missing local branch errors first (`parenttips.go:54-56`). If it did happen, the result would be `false, nil`.
  - Case: the first tip is clean and the second conflicts, so the result is `true`.
  - Case: `ConflictsWith` errors, so it returns `false, err` and that error is not taken as a conflict. Correct.
- **`Fabric.ConflictsWith` (`parenttips.go:222`)**
  - Case: ref is a parent tip SHA that head already contains. `MergeTree` exits 0, so `clean = true` and the result is `false`. Correct.
  - Case: both sides edited the same file. `MergeTree` exits 1 and returns `clean = false` with a nil error (`gitrepo/merge.go:343-344`), so the result is `true`. Correct.
  - Case: any other git failure. The default branch returns an error, which is wrapped and kept apart from a conflict. Correct.
  - Case: ref does not resolve. The error names the ref. Correct.
  - The HEAD it merges into is the beam pair's code HEAD, which `fabricengine.Open(beam)` opens.
- **`judgeOpenable` (`pr.go:241`)**
  - Case: `Tasks` errors. It returns `rerunRefusal` for "read the board". Correct.
  - Case: the merge task `b-merge-in` has a nil status. `IsDoneStatus(nil)` is false, so it refuses. Correct.
  - Case: the merge task has status done. `openMergeTask` skips it (`mergeparent.go:101`).
  - Case: another beam's merge task. The `Project == beam` filter excludes it.
  - Case: a non-merge member of this beam. `IsMergeTaskSlug` excludes it.
  - Case: `ParentConflicts` errors, for example on a failed fetch. It returns a rerun refusal before any push. Correct.
  - Case: no conflict and parent only ahead. It returns nil and the open proceeds. Correct.
- **`prOpen` with no pull request recorded**
  - `judgeOpenable` runs after `requireOrigin` and before `PushBeam`, `FindOpen`, `SetDraft`, `Create` and `WriteLanding`.
  - On refusal nothing has changed. Correct.
- **`openRecorded`**
  - Closed state: `judgeOpenable` runs after the read-only `Status` call and before `Reopen`, so a refusal changes nothing. Correct.
  - Open state: no check runs. It is not an open, so this is out of scope for my lens.
  - Merged state: unchanged.
  - Unknown state: unchanged.
- **`TestPROpen` check for untouched state on refusal (`pr_test.go:256-266`)**
  - When `wantUntouched` is false it returns early. When true it asserts zero calls to Create, Reopen, SetDraft and Edit, zero pushes and zero writes. Correct.

**Findings:**

1. **NIT, consistency.** `internal/fabricengine/parenttips.go` `ConflictsWith`: when `ResolveSHA("HEAD")` fails, the error is wrapped as "conflict test against &lt;ref&gt;". That names the parent ref even though HEAD is what failed to resolve, so the message misleads. Evidence: the diff hunk at `parenttips.go:226-229` (around the `headSHA` resolve).
2. **LOW, consistency.** `internal/beamcli/deps.go` `ParentConflicts`: it throws away the head that `beamParentTips` already read, and `ConflictsWith` resolves `HEAD` again. Each call therefore reads HEAD twice. The two reads see the same HEAD, so the result is unaffected, but the seam reads more than it uses. Evidence: the `deps.go` diff, `ParentConflicts` at `pair, _, tips, err := beamParentTips(slug)`, and `parenttips.go` `ConflictsWith`, which resolves "HEAD".

**Tool calls:**

- Read `ReadParentTips` and `HeadSHA` in `internal/fabricengine`, to confirm that tips are SHAs, that the local tip is always present, and that `HeadSHA` reads the code HEAD. My context had only their call sites.
