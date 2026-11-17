```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test row that pins it.

**Behaviours and the tests that pin them**
1. `sheddriver.handOver` hands over without reading the tree (the dirty-tree check is removed). It is pinned by `internal/sheddriver/drive_test.go` `TestGoDriver_Settle`, in the row "a changed build records the hand-over, parks, notifies before releasing the lock, spawns and exits" at about line 574. That row asserts the calls `step`, `release` and `spawn`, the order `notify`, `release`, `spawn`, the action `SettleHandOver` and the build `newbuild1234`. The driver now has no seam that reads the tree, so a dirty tree is not an input the unit can take. A guard added back would need a new `HandOverSeam` field and a new fake. The row would then have to change, so the behaviour is held in place by the seam's shape.
2. The hand-over still runs once per stop key and build. `handedOver` is unchanged, and the existing `TestGoDriver_Settle` hand-over rows still cover it. Nothing in this diff changed it.
3. The old dirty-tree hand-back, with its report naming the paths and no spawn, is removed along with its row. No remaining test asserts that removed text or the `DirtyPaths` hand-over seam. I grepped `*_test.go` files for "not clean for a hand-over", "uncommitted paths outside the run's records", `dirtyPathsOutsideRecords` and `DirtyPaths:`. The only hits are other packages' unrelated seams: `landingshed`, `shedrecipe`, `fabricengine` and `verifytree`.
4. Wiring: `loomcli.goDriverDepsFor` drops `DirtyPaths` and `dirtyPathsOutsideRecords` is deleted. A removal adds no behaviour, so it needs no test, and the package builds.
5. The doc and comment rewording in `internal/loomcli/doc.go` and the `handOver` doc is prose, which the lens exempts.

**Findings:** none.

Tool calls:
- One grep of `*_test.go` for the removed hand-back message, the `dirtyPathsOutsideRecords` helper and `DirtyPaths:` fakes. My context did not show whether any other test asserted the removed behaviour.
