```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the hoisted template copies and the moved `boardkit.InstallReplay` in `worktreeseam_integration_test.go`.

No findings.

I ran each changed path by hand:

- **Copies before the seam:** `boltOnlyOrigin`, `failingPullOrigin`, `parentPairOrigin` and `reconcilePair` are taken at the top level, before `InstallWorktreeSeam`. The first build of `originTemplate` or `movesTemplate` therefore never runs under the fake seam, whatever `-run` filter is used. The top-level copies run regardless of the filter, so every later subtest finds its copy ready.
- **Moved `InstallReplay`:** it now runs before the `movesTemplate` copy, as it did before the fix. The `movesTemplate` build sees the same replay state as in `TestReconcileOrigin_Moves`. Its cleanup runs after the seam's cleanup (last in, first out), which is a safe order.
- **Subtests' assertions:** the Bolt-pull and weft-only subtest asserts no events under `origin.Path`, and the hoisted copy fires none because it is made before the install, so that assertion still holds. The failing-pull and parent-pair subtests call `seam.failWithin(origin.Path)` after the copy, so the failure still applies to the events they check. The reconcile subtest still has its `pair.Path` events recorded only after the install.
- **Copies on the parent `t`:** the subtests are not parallel, so the parent's `t.TempDir` and `t.Cleanup` hold the hubs until the whole test ends. This leaks nothing and breaks no ordering.
- **Compile check:** every new variable is used, and no name clashes with the existing `hub`, `fab`, `second` or `seam`.
- **Gate:** the tier-1 gate on the package passed in the parent's context.
- **Regression coverage:** the fix is a setup-order change and adds no test. The existing subtests would not fail if the ordering regressed, but the review asked for no new test.

No further tool calls were made.
