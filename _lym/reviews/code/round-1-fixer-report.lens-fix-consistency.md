```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix for F1 and F2 for consistency with its surroundings (lens fix-consistency).

Findings:

- **LOW** — `internal/fabricengine/housekeeping_integration_test.go`, the doc comment of `TestStoreHousekeeping`.
  - The comment says the test "drives Add's key writes and Remove's foreground gc over hubs from hubforge", and the next sentence says each subtest unsets the keys "right before the step whose write it checks".
  - The new subtest `NewTopology starts at git's default gc.auto` drives neither Add nor Remove, builds no hub, and unsets no keys. It only reads `NewTopology(Config{})`.
  - A reader of the comment would not expect it. One added clause would fix this, for example "It also pins the threshold `NewTopology` starts with".

No other disagreement found:

- `TestSyncOptionsNotifier` follows the sibling convention: it sits in `remoteteardown_test.go`, calls `tiergate.Require(t, 0)`, is serial, states why in its doc comment, and uses `recordNotices` as its siblings do.
- `GCAutoThresholdForTest` sits beside `SetGCAutoThresholdForTest` and matches its naming and comment style.
- The `export_test.go` header covers the new accessor.
- The new `TestSyncOptionsNotifier` leaves its sibling tests unaffected: it uses `recordNotices`, which restores the previous notifier on cleanup.

Further tool calls: none. My context already held every changed file.
