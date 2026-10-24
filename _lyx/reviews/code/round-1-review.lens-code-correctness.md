```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the amendment, which is the parent's merge into `docs/overview.md` and `internal/landingshed/doc.go`.

**Per function or branch, case run, result**

- `internal/landingshed/doc.go` is a comment-only file. The task adds the `recipe-owns-failures` sentence at line 7. The merge keeps the parent's "first failing tier" wording at line 123. Neither is an executable branch, so I ran no input. No defect.
- `docs/overview.md` is prose. The merged gate line (line 461) carries the parent's new text: a failing tier stops no later tier or guard run, and the call exits with the first failing run's code. It keeps the task's `PATTERN-` regrouping elsewhere in the file. No code path to run. No defect.
- The task's amendment-range diff, `5e36064a3..HEAD`, changes no Go function. The `internal/gatecli`, `internal/gateslot` and `verifygate.go` changes belong to the parent and are outside the amendment ("parent changes a Merge-In brought are left out"), so I judged none of them.
- The `PATTERN-x` hits in `internal/landingshed/mergerecord_test.go:100` and `:102` are an `Overlap` classification fixture, not a citation of a removed ID. Not a defect.

**Findings:** none.

**Tool calls:** none. My inherited context held the merged diffs and the doc text.
