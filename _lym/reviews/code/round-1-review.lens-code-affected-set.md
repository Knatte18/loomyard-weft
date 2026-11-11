```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's changes force outside the lines it touches.

**Per change, the set found:**

1. **`weftConflictStop` extraction (`pull.go`)**
   - The only production caller is `settleUnmovedWeft`, `pull.go:439`.
   - The extracted function builds the same `WeftStopError` as the removed literal: Branch, Paths, WayForward and Err all carry over. `outcome.Conflicts` becomes `paths` and `err` becomes `cause`.
   - The `recordsConflictWayForward` and `lymVerbDir` calls are unchanged.
   - The new `WeftConflictStopForTest` in `export_test.go` pins it, and it mirrors the existing `WeftDirtyStopForTest`.
   - No doc, stencil or PATTERN entry names the inline literal.
   - Covered, no finding.

2. **`newFailureFixtureAt` and `newPairFixtureAt`**
   - Every other caller (the `newFailureFixture` and `newPairFixture` call sites, `pushclasses_integration_test.go:194` through 797) still goes through the wrappers with the anchor `"."`, the old default.
   - Their behavior is unchanged.
   - Covered, no finding.

3. **Anchor-change table gains the `hubAnchor` field**
   - The `WhileTheOwnMarkerIsEmpty` and `WhileAMoveRecordStands` rows keep `"."`, so what they test is unchanged.
   - The renamed first row's expected string is the full anchored way forward in `warpPath/backend`.
   - The only code that asserts these names is the loop in the same test; no other file refers to the old row name.
   - Covered, no finding.

4. **PATTERN and docs**
   - The diff changes tests plus one internal method with no new file, lock, git call, stencil or refusal text.
   - I found no PATTERN entry or `pattern/` background file that the diff touches or forces a change in, so there is no landing to check.
   - The commit has no `Pattern-Departure:` trailer, and none is needed.
   - Covered, no finding.

**Findings:** none.

**Tool calls (1):** one `grep` for callers of the fixture helpers and the anchored variables in `failedpush_test.go`. My context lacked the callers of `newFailureFixture` and `newPairFixture` elsewhere in the test file. The `hubforge` path glob in that call matched nothing, so I did not verify `Shape.Anchor` handling in `hubforge` or whether `anchoredTask`, `anchoredWeft` and `anchoredDir` are in scope for the new row. That check belongs to the handler.
