```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fixes for F1–F4 cover every member of each finding's set.

No findings.

- **F3 (marker-position rows):** `make_change` is the only Go-filled marker anchored under a heading in `darn-template-write.md`. The others (`parent_directive`, `edit_directive`, `pattern_directive`, `project_directive`, `friction_directive`) sit before Step 1, under no heading. The new row therefore covers the whole set.
- **F1 (callers of `GateFixed`):** `DarnInputs.GateFixed` is set only in `darnwiring.go:256`, and the new wiring test exercises that path. The test covers all three branches: pair slug with a fixer, pair slug with an empty fixer, and no pair slug with a fixer.
- **F2 (fragment stencils):** `makeChangeStencil` and `makeChangeGatedStencil` are the only two fragments `makeChangeList` reads. Both now have a missing-fragment row.
- **F4:** it names fixed lines in a single row, so there is no set to check.

Unchecked: none.

Tool calls:
1. One `grep` of the markers and headings in `darn-template-write.md`, plus `GateFixed`, `makeChangeList` and `DarnInputs{` across the Go files. My context lacked the stencil's marker list and the full list of `DarnTable` callers.
