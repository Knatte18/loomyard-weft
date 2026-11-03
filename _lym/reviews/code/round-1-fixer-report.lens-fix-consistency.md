```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- **Stencil strings:** the new literals in `wiring_darn_test.go` match the fragment text at `darn-fragment-make-change.md:11` and `darn-fragment-make-change-gated.md:11`. They also match the `darnengine` constants in `spec_test.go`.
- **F3 row:** the heading in the new `markers_test.go` row matches the others in that table. The changed packages' tier-0 gate passed.
- **F2 rows:** both new rows name the removed stencil constants (`makeChangeStencil`, `makeChangeGatedStencil`), the same way the existing missing-stencil row names `darnStencilName`. The reworded comment now agrees with `makeChangeList` in `spec.go`.
- **Duplicated literals (LOW, not a finding):** the loomcli test and the `darnengine` test each hold their own copy of the two step lines. This follows from the package boundary, because the constants are unexported. A stencil rewording would fail both tests loudly, so nothing is left stale.

Tool calls:
1. `git diff --stat` plus a grep of the fragment text, to confirm the diff covers only the three test files and that the literals match the stencils on disk. My context had the test files but not the fragment text.
