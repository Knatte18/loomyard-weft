```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Per behaviour, the test that pins it**

1. Template profiles speed, light, solid and wide hold a jacquard block equal to tapestry minus `loom.discussion`, and docs holds none. Pinned by `TestTemplateLadder`'s new jacquard code and plan rounds, its no-discussion and `len==2` check, and its reviewer_fan `blockKeys` row (`internal/runprofile/runprofile_test.go:453-473`). Also pinned by `TestReinterpretJacquardBlocks_ShippedTemplateUnchanged`.
2. Template `defaults.jacquard: solid`. Pinned by `TestLoad`'s "no file" row (`runprofile_test.go:48`), by `TestTemplateLadder`'s `Default(jacquard)` (`:434`) and by boardcli `stepProfiles` (`verbs_test.go:1077`).
3. Registration after the arras reinterpretation, with a non-nil Rewrite. Pinned by `TestLookup_ProfilesCarriesTheJacquardBlocksReinterpretationAfterTheArrasOne` (`configreg_test.go:27-45`).
4. A tuned tapestry block replaces a differing jacquard block, written in place, keeping the other loom areas. Pinned by row "a shipped profile's tuned tapestry block…" (`reinterpret_test.go:36`).
5. An operator-added profile gains a block by append. A loom module whose only area was discussion is dropped. Comments are kept. Pinned by row "an operator-added profile…" (`reinterpret_test.go:48`).
6. A jacquard block equal to the derived one is left as is, and `defaults.jacquard` follows a changed `defaults.tapestry`. Pinned by row "a changed tapestry default…" (`reinterpret_test.go:55`).
7. `defaults.jacquard` is dropped when `defaults.tapestry` is absent. Pinned by row "a jacquard default is dropped…" (`reinterpret_test.go:67`).
8. An aliased profile is written into its anchored mapping. Pinned by row "an aliased profile…" (`reinterpret_test.go:78`).
9. A profile supplied through a root merge key is flagged, and a default naming a profile with no jacquard block is dropped. Pinned by row "a profile supplied through a merge key…" (`reinterpret_test.go:87`).
10. A file that already matches comes back byte for byte with no line. Pinned by `TestReinterpretJacquardBlocks_ShippedTemplateUnchanged` (`reinterpret_test.go:132`).
11. The arras rewrite now goes through the shared `encodeDocument`. Pinned by `TestReconcileHubWideAt_ProfilesStillLoadAfterTheArrasBlocksDrop` (`internal/configsync/configsync_test.go:1143`, wantLines "defaults.arras: light is dropped").
12. `defaults.jacquard` supplied through a merge key is flagged, on both the set and the drop path (`reinterpret.go` `reinterpretJacquardDefault`, `mergeFlag` returns). **None.**
13. An aliased `defaults` entry is rewritten in its anchored mapping (`ownMapping` on `DefaultsEntry`). **None.**
14. A file whose only lines are flagged comes back unchanged, with the flags (`reinterpret.go` `if len(rewrote) == 0 { return existing, nil, flagged, nil }`). **None.** The only flag row also rewrites the default.
15. A profile holding a jacquard block but no tapestry block is left untouched and counts as holding jacquard for the default. **None.**
16. The new reinterpretation run by the reconcile after the template follow, over a hub file and base from the pre-jacquard template. **None.** Every row is hand-built post-follow input. The arras change has such a test (`configsync_test.go:1143`), and the configreg test's `//testtiming:keep` note says no walk test reads the order.

**Findings**

- **MEDIUM, design.** `internal/runprofile/reinterpret.go`, `reinterpretJacquardDefault`. The merge-key flag on `defaults.jacquard` (behaviour 12) has no row on either path. Removing either `mergeFlag` return, or swapping it for a silent no-op, fails no test. Evidence: `reinterpret_test.go:36-94` has no row with `defaults` under `<<` or holding jacquard through a merge.
- **LOW, design.** `internal/runprofile/reinterpret.go`, `ReinterpretJacquardBlocks`. The defaults-alias path (behaviour 13) and the flag-only path (behaviour 14) are uncovered. Evidence: `reinterpret_test.go:36-94`; the merge-key row at `:87` also yields a rewrote line, so the early return at `reinterpret.go` `if len(rewrote) == 0` is reached with flags by no row.
- **LOW, design.** `internal/runprofile/reinterpret_test.go`. No test runs the reinterpretation after the real follow (behaviour 16). The acceptance criterion that a tuned shipped profile gains a carried jacquard block "at reconcile" is pinned only on input that already holds a jacquard block. A regression in how the follow-added block meets the rewrite, such as the follow's flow-style copy or the base's template-base, would pass. Evidence: `reinterpret_test.go:38` pre-seeds `jacquard:`; the sibling pattern is at `configsync_test.go:1143-1174`.
- **LOW, design.** `internal/runprofile/reinterpret.go`, profile loop. A profile holding a jacquard block but no tapestry block (behaviour 15) has no row. Breaking the `continue` on `!ok`, or `holdsJacquard[name] = held`, would go unseen. Evidence: `reinterpret_test.go:67-77`, where the docs profile holds neither block.
- **NIT, consistency.** `internal/runprofile/runprofile_test.go:434`. The new `Default(jacquard)` assertion repeats coverage that the updated `TestLoad` "no file" row already holds at `runprofile_test.go:48`: the template's `defaults.jacquard` is solid. It matches the existing motif pattern, so it is no new test, but it adds no coverage.

**Tool calls**

1. Grep for tests touching the arras reinterpretation, and for jacquard in reconcile tests: needed to know whether the `encodeDocument` refactor and the walk path had any test, which my context lacked.
2. Grep in `configsync_test.go` for arras and the reinterpretation IDs: needed to confirm that the configsync test exercises the arras reinterpretation's output line.
