```yaml
lens: fix-consistency
outcome: reported
```

Fix-consistency lens: the F1 change set is consistent with its surroundings except for one stale sibling in the spec.

**Findings**

- **LOW** — `contracts/specs/raddle-proposal-spec.md:40`, the `labels` field description.
  - The line says only "at least one, each a module label of the board's vocabulary". It does not say each label may be listed once.
  - The new `label-duplicate` check at line 106 now refuses a repeat, so a reader of the field contract would not learn the rule from the field itself.
  - Suggested edit: add "listed once" to line 40, for example "at least one, each listed once and each a module label of the board's vocabulary".

**Checked and consistent**

- `CheckLabelDuplicate` sits beside `CheckLabelUnknown` in the `check.go` constants.
- The `vocabulary` doc comment and the `label-duplicate` spec bullet at line 106 say the same thing.
- The project's labels are already deduplicated by `raddleProjectLabels` at `internal/loomshed/raddlefile.go:532`, so no twin check is needed for the project.
- No other file lists check ids: `grep` for `label-unknown` and `labels-missing` hits only the spec, `check.go` and `check_test.go`, and the `_bolt` stencils have none.

**Further tool calls**
1. `grep -rln` for `label-unknown`, `labels-missing` and `CheckLabelsMissing` across the worktree and the `_bolt` stencils, to find any other list of check ids; my context lacked it.
2. `grep -n "label"` on the spec plus a grep for "more than once" in the docs, to find sibling wording about labels; my context held only lines 100–110.
3. `grep -n "Labels"` on `internal/loomshed/raddlefile.go`, to see whether the project's label aggregation could repeat a label; my context lacked that file.
