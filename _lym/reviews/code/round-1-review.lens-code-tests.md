```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour: pinning test**

- `registerVerbTemplates`, `verbTemplateKey` and `copyVerbHub` (one template per verb and anchor, copied per cell). The cross-product cells in `runCell` (`livestate_matrix_test.go:258-266`) pin it. A wrong key or a missing registration makes `verbTemplates[...]` nil, so `CopyRegistered` fails every Remove, Prune, Cleanup, Checkout and UnwireJunctions cell. `TestVerbCases_ArrangeSetsItsPreconditionField` also runs `copyVerbHub` for Add, Checkout and Pull.
- Remove, Prune, Cleanup, Checkout and UnwireJunctions Build/Arrange split. Each verb's own `Expect` effects pin it. These check that the pair's worktrees are gone for Remove and Prune, that the orphan weft branch is deleted for Cleanup, and that the branch is switched for Checkout. Both halves have to run for the cells to pass.
- Cleanup's `OriginalBranch`, now read from `BoardDir()` (`livestate_verbs_test.go:642`). The Cleanup effect (`:666-669`) asserts that the weft branch `RecordsBranchName(f.OriginalBranch)` still exists. A wrong value would make that assertion fail.
- Checkout's `OriginalBranch`, read after the Build. The refused-Checkout effects (`:726-730`) and the proceeding effect (`:776`) pin it.
- `slug` as a shared constant. The `PermittedRoots` and `Effect` checks in each case pin it. A mismatch would show up as an unpermitted manifest change or a failed `assertGone`.
- Add, Pull, Reconcile, the hostile cases and CloneHub{Reset} keeping their Arrange whole, with no `Build`. The same cross product pins it through the `Build == nil` branch of `copyVerbHub`. The CloneHub{Reset} column, `livestate_matrix_test.go:382`, still calls `CopyHub` plus `Arrange` directly, which is correct while those cases have no `Build`.
- The six merge layers (`conflictLayer`, `clashLayer`, `nonFastForwardLayer`, `warpProgressLayer`, `featureBothSidesLayer`, `cleanMergeableLayer`). Each one's consumer tests pin it: the MergeIn conflict, abort and continue tests, the recovery tests, the lock tests, the guard tests and the crucible tests. A wrong layer commit would change those results.
- `newMergeLayerFixture` and `mergePairFixtureOf`. Every consumer above exercises them.
- `commitOnBranch`, `branchAtCurrentHEAD`, `setupConflictingDivergence` and `setupCleanNonFastForward` moving from `*testing.T` to `testing.TB`. This is a signature change only, and the layer builds and the unchanged sites cover it.
- The registration's build constraints (no Setenv, board at the upstream tip). `hubforge`'s own template tests pin these, and they are outside this diff.

**Findings**

- **None for missing coverage.** The diff adds no production behaviour. Every moved fixture is exercised by the tests that consume it.
- **NIT, class `consistency`, `livestate_matrix_test.go:382-383`.** The CloneHub{Reset} column builds its hub with `hubforge.CopyHub` and calls `verbCase.Arrange` directly instead of using `copyVerbHub`. No case in that column has a `Build`, so nothing breaks. A `Build` added to one of those cases later would be skipped silently, and no test would catch it.

**Tool calls**

I made none: my inherited context already held everything this lens needed.
