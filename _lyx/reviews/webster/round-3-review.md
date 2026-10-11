---
verdict: APPROVED
---

No findings.

The Publish verify failed with `concurrent map writes` in `internal/clihelp`.
The stack in `.lyx/verify/verify-4.log` runs from `visibleLocalFlags` through `cobra.(*Command).LocalNonPersistentFlags` and `mergePersistentFlags` into `pflag.(*FlagSet).AddFlag`.
Parallel subtests of `TestRenderIndex_FormsAndFiltering` shared one `root`, so cobra's lazy flag merging wrote the same `pflag.FlagSet` maps from several goroutines.

Commit `321e93b13` removes the `t.Parallel()` call from the `t.Run` loop at the end of `internal/clihelp/index_test.go`, so the subtests run one after another over `root`.
The parent test keeps its own `t.Parallel()`, which is safe because it builds a private `root`.
It states why the subtests are serial in a comment above the loop, which the test-speed pattern allows for a shared-state test.

Coverage is unchanged.
The `tests` table still has one row per audience, and each row is asserted with the same exact-equality `RenderIndex` comparison.

Other parallel subtests in `internal/clihelp` (`jsonhelp_test.go`) only read an already-unmarshalled result, or build a fresh tree per row, so none share a tree.
The two `RenderIndex` callers in `internal/orchcli` call it on their own harness's root, outside any parallel fan-out over one tree.
