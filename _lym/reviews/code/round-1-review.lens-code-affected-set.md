```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens on the merge's conflict resolutions (`bae0ec63`) and commit `382a591f`.

**Affected set, per change**

- **`internal/beamcli/pr_test.go`: `wantNot` removed, the handed-over row moved to `wantNoErr`.** Both fields belong to the `TestPROpen` table and are read only by its loop.
  - No `wantNot` is left in `internal/beamcli`. Nothing else reads the field, so the set is covered.
  - `wantNot []string` in `internal/loomshed/verifygate_test.go:42` is another test's table field and has no tie to this change.
  - Covered.
- **`internal/lymdirs/doc.go`: the package doc merges the `HandOverDirName` sentence with the `UsageDirName`, `RaddleDirName` and `WeaverDirName` list.**
  - Each of the four constants exists, and each constant's own doc agrees with what the package doc says of it (`internal/lymdirs/dirs.go:54`, `:90`, `:94`, `:99`).
  - `PATTERN.md` and `pattern/` do not name these constants. `pattern/PATTERN-run-fsm.md:46` mentions a hand-over mark under the hub's `.lym`, which agrees with `HandOverDirName`.
  - Covered.
- **`internal/reedengine/bindings_integration_test.go`: three literal 30-second waits become `recorderDeadline`.**
  - The constant is defined at `:412` with the same value, and the `time` import is still used. No other file reads the literals.
  - Covered.
- **`internal/heddleengine/overview.go` and `overview_test.go` (`382a591f`): only comments change.** No signature, output or behaviour changes, so nothing outside these lines depends on them.
  - `lym loom lint-comments --base 0c013afa…` is clean at HEAD.
  - Covered.
- **Rubric checks this lens applies:**
  - Commit messages and departures: neither `bae0ec63` nor `382a591f` carries a `Pattern-Departure:` trailer. The trailer text the grep found comes from the parent's commits, which are no subject. No finding.
  - Declared PATTERN changes: the directive declares none, and the `PATTERN.md` and `pattern/` changes in the range arrive with the parent's commits. No finding.
  - Stencil wording: no stencil is touched. Not applicable.

**Findings:** none.

Tool calls:
1. One Bash call grepped `PATTERN.md` and `pattern/` for the `lymdirs` names, grepped the code for leftover `wantNot`, grepped the range's commit messages for `Pattern-Departure` trailers, and took the range's PATTERN diff stat. My context had not searched the rule documents or the rest of the repository for these names.
