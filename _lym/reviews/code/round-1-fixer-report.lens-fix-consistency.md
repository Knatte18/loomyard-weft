```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change in `unwire_test.go` (new `wantStripped` field and assertion) for consistency with its surroundings.

**Findings**

- **LOW** — the doc comment of `TestUnwire_LeavesSiblingWorktreeUndirtied` is now stale.
  - Changed text: the paragraph "The row then asserts that the prime and the kept pair report the same git status as before, and that the kept junction lives."
  - The test now also asserts, for rows with `wantStripped` set, that the unwired pair's anchored exclude line is gone from the prime's exclude file.
  - A reader of the doc comment would not learn that the test pins the strip side of the wiring.
  - Evidence: the new block `if tt.wantStripped != "" { ... containsLine(lines, tt.wantStripped) ... }` after the `fslink.IsLink` check. The comment above the test, at `internal/fabricengine/unwire_test.go:325-327`, names only the status comparison and the kept junction.

Nothing else disagrees with the change:
- The new struct field comment matches its use.
- The three rows' values fit `excludeLinesToStrip`. Only `UnwiredPairAloneAtItsAnchor` has a kept pair at a different anchor, so only its `/sub/_lym` is stripped, and the tier 1 gate passed.
- The planted bare `_lym` line in that row stays because the kept pair wires `_lym` at another anchor, so it does not contradict the new assertion.
- `containsLine` and `gitkit.ExcludeLines` were already used in this package and file, so the usage matches.

Tool calls made: none. My context already held every file the lens needed.
