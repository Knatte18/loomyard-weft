```yaml
lens: fix-code-correctness
outcome: reported
```

**no findings**

- **F1 (`writerFanGaps` moved out of the `emphasisLineGaps` closure):**
  - The moved block uses only `writerOf`, `tests` and `tt`, all defined inside it. It never used the outer closure's `exempt` or `tests`, so nothing is left dangling.
  - `t.Parallel()` is still first in the moved subtest.
  - The closing braces balance, and `writerFanGaps` is a direct child of the parent.
  - The gate run passed on `./internal/lenses`, and the subtest name is now `writerFanGaps`.
  - No test was added, so no test needs to fail with the defect in place.
- **F2 (assertion reduced to `procedureLenses["leanness"]`):**
  - Line 401 is still needed: the map is passed as the exemption set to `emphasisLineGaps(cfg.Lenses, procedureLenses)`.
  - With `leanness` wrongly marked as a procedure lens, the assertion still fires. The error message matches the check.
- **Commits:** the F2 commit was made from a clean checkout plus the F2 edit alone, so it compiles on its own.

Tool calls beyond the context: none.
