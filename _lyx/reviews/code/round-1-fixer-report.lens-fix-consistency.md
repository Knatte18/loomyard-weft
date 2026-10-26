```yaml
lens: fix-consistency
outcome: reported
```

Fix-consistency lens: twins of the fixer's verb-list edits and the other changed text.

**LOW — `internal/websterengine/runlevel.go:777`: stale verb list, a twin of F7 left unchanged.**
- The comment reads "Master backgrounds recover-batch and fix-verify and recovery_timeout_min bounds both, so the wait marker leaves them out."
- The fixer added `fix-plan` to the same kind of list in `wiring.go:344`, `batchpause.go:1`, `batchpause_test.go:3` and `cli.go:17`.
- `runlevel_test.go:1593` is named "the recover-batch, fix-verify and fix-plan shells are declared awaited", and it checks `masterAwaitedShellPrefixes`, which this comment describes.
- The comment now disagrees with that test and with the other edited lists.
- I did not read `masterAwaitedShellPrefixes` itself, so this rests on the test name.

No other findings:
- The `roleOpeningStencils` entry `plan-fixer` and the `markers_test.go` row for `webster-template-plan-fix` agree with the stencil's markers as the stencil header documents them.
- `Cause` is gone from `ConflictRecord`, and `doc.go` has no reference to `Cause`.

Tool calls:
- A grep for the verb-list phrases across `.go` and `.md` files. My context showed only the lines the fixer touched, not their twins.
- A grep of `internal/websterengine/doc.go` for `ConflictRecord` and `Cause`. My context did not show whether the docs mentioned the removed field.
- A grep of the stencils, `PATTERN.md` and `docs` for `plan-fixer` and `verify-fixer`. My context did not show whether other tables list the roles.
- `sed -n 770,785p` plus a grep of `fix-plan` in `runlevel.go`, to confirm the stale comment's wording and whether `runlevel.go` mentions `fix-plan` elsewhere.
