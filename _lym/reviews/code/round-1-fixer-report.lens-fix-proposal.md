```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals of F1, F2 and F3, which the fixer applied as proposed.

**F1 (`pattern/PATTERN-test-speed.md`, the reworded `lym gate testmap` bullet)**
- LOW: the new text says a non-prime worktree's maps lie "under its `.lym`". F2 shows they lie under the anchor's `.lym` (`coverageMapsDir` returns `testcover.Dir(location.AnchorPath())`). With a subdirectory anchor the sentence points at the wrong directory.
- The reviewer's proposed wording caused this, and the fixer kept it. The test comment was corrected to "its anchor's `.lym`", so the PATTERN bullet now disagrees with it.
- The same loose "under .lym" appears in the `testmap` Long.
- The edit also left a colon and a trailing comma in the bullet's one sentence ("…it runs in: …, a worktree's own maps under its `.lym` elsewhere, and the `redundancy` report step…"). It parses, but a reader may take "and the `redundancy`…" as a third map location.

**F2 (`TestGateTestMapInATaskWorktreeKeepsItsOwnMaps`)**
- No defect found. A `coverageMapsDir` that returned the worktree root now fails, because `readTestMap(anchor)` would find no map.
- The test still asserts nothing about the root: it never checks that `testcover.Dir(pair)` holds no map. A regression that wrote to both places would pass.
- This is not a finding, because the review asked only for the anchor pin.

**F3 (`testmap` Long)**
- LOW: to stay under 2000 bytes, the fixer deleted "and run it over the whole module before a sweep of redundant tests". The review proposed folding only "It runs in any worktree of the hub."
- The operator guidance to build over the whole module before a sweep is gone from the help, though the review did not ask for that cut.
- The doc still carries it. The `lym gate testmap` PATTERN bullet does not cover it, though (`PATTERN-test-speed.md:287-294`).
- The fixer also cut "of the package" from the "per-test blocks" clause and "also"/"package's" from the named-packages line. These are harmless.
- The audit sentence matches the old text and the live audit behaviour (`testMapAuditTests`).

**Tool calls:** none. My context already held every changed file and the diffs.
