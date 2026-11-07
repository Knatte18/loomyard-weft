```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens, meaning whether the change set still agrees with the docs, comments and sibling code it touches.

**Findings**

1. **LOW** — `internal/fabriccli/doc.go:23-24`, the clone verb's package doc.
   - Summary: it was left as it was, but a clone's warning no longer always reaches `seam_warning`.
   - Evidence: the doc says "A failed render of heddle's files into the prime never fails the clone: its warning is the envelope's `seam_warning`".
   - The F2 change in `clone.go` and the `fabricengine/doc.go` addition say that a walk failure after a failed render returns the warning in the error instead.
   - A reader of the verb doc would expect `seam_warning` on every failed render.
   - The twin should carry the same exception: "unless the hub walk then fails, which carries it in the error".

2. **LOW** — `internal/fabricengine/clone.go:84-86`, the `CloneResult.SeamWarning` field doc.
   - Summary: the field doc says nothing about the walk-failure case, where the field is lost (the function returns `CloneResult{}`).
   - Evidence: the doc says it is "the warning text of a failed worktree event for the prime".
   - `CloneAndWire` now returns an empty result with the text folded into the error on a walk failure.
   - This is minor, since the `CloneAndWire` doc and `fabricengine/doc.go` now state it.

The other twins agree with the change:
- The `Made` doc in `seam.go` matches its new behaviour.
- The TookIn path still logs only, which the F3 `fix` allowed.
- The `runCloneWithReset` envelope comment on `seam_warning` ("a failed render into the prime") still holds, since F3 routes warn-only renders into the same field.
- The `fabricengine/doc.go` sequence claim agrees with the `clone.go` comment.

**Further tool calls (2)**
- A grep of `heddleengine/doc.go` and `fabricengine/worktreeseam.go`, plus a repo-wide grep for `seam_warning` and `SeamWarning`. My context lacked the other places that describe the seam warning.
- A read of `fabriccli/doc.go:18-30` and `fabricengine/clone.go:80-88`, to see the exact text of the two twins the grep found.
