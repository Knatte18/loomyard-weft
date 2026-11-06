```yaml
lens: fix-affected-set
outcome: reported
```

Findings: none.

- F1 (stencil routing): `plan_order` appears in `contracts` only in `webster-template-master.md`, lines 238, 240 and 242, and all three agree. `plan_defect` and `tree_moved` appear in no other stencil. No sibling stencil still routes `plan_order` through `fix-plan`.
- F2 (quoted way-forward text): the string "edit the plan so the named cards match the tree" now survives only as the `plan_defect` default at `internal/websterengine/beginbatch.go:115`. No stencil or doc quotes it.
- F3 (check-name literal): the only remaining `"delete-before-reference"` literals in Go are these.
  - `internal/webstercli/cli_test.go:268` asserts the wire output of the CLI envelope. A literal there is deliberate, and it is outside the review's two named sites.
  - Two test-name strings in `internal/planglyph/glyphchain_test.go` (lines 962 and 992) are not comparisons.
  - The constant definition itself.

Unchecked: nothing.

Tool calls:
1. Grep for `"delete-before-reference"` literals in Go files. My context lacked a repo-wide search for the F3 site set.
2. Grep for `plan_order` and the old way-forward quote in `contracts` and `internal`. My context lacked a repo-wide search for the F1 and F2 site sets.
3. List files containing `tree_moved` or `plan_defect`. I needed to know whether a sibling stencil enumerates the causes.
