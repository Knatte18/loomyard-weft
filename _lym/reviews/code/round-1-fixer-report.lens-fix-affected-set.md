```yaml
lens: fix-affected-set
outcome: reported
```

No findings on F1, F3 or F4. The one finding is on F2 and F3's sibling text.

**LOW** — `internal/boardcli/doc.go:68`
- The package doc still says the P0 row is "placed before the first project or note".
- F2 changed the rule to before the first project, note or done entry.
- F3 changed it to last under `--sort`.
- The fix updated the `list` help text in `cli.go` and the `withRootProjectRow` doc comment, but not this twin. Evidence: `doc.go:67-68` against `text.go:89` and the new `cli.go` list help text.
- The same lines also say the `--sort` rule, "An entry lacking the date comes last", at `doc.go:70-71`. That rule now needs to say the root row also goes last there.

Unchecked, with no finding:
- F1's set (`list --text` runs with `--label`): the fix adds the one missing assertion.
- F4's set (comments that say `parent_beam` is empty for the prime):
  - `cli.go:1376` and `boardengine/doc.go:169-172` already say the listing prints `root`, or describe the empty slug the resolver returns.
  - `parentbeam_test.go:151` is a seam-failure case, where the cell is empty by design.
  - `docs/research/.../report.md` is a measurement report, so it is exempt.
- F2's done-entry set: `withRootProjectRow` has one caller, and `BriefTask.IsDone` has no other sites.

Tool calls:
1. `grep` for `withRootProjectRow` and `RootProjectBrief` across `.go` and `.md` files, then `grep` for the empty `parent_beam` wording and for "project or note": to find twin sites that the context lacks.
2. `sed` on `boardcli/doc.go:64-72`, `cli.go:1374-1380` and `boardengine/doc.go:166-173`: to read the twin wording at the hits.
