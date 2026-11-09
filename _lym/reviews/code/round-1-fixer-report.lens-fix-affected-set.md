```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Task: fix-affected-set lens. I reran the sets behind F1 and F2 against the changed tree.

- **F1** (callers of `SyncOptions.notifier`): the only production call is `pushclassified.go:283`. The new `TestSyncOptionsNotifier` calls `notifier()` directly and covers both branches: nil `Notifier` falls back to the installed notifier, and a set `Notifier` shadows it. The set is covered.
- **F2** (constructors and readers of `gcAutoThreshold`): `NewTopology` at `topology.go:37` is the only `Topology{` literal. The only read is `remove.go:420`. The new `GCAutoThresholdForTest` assertion is on `NewTopology(Config{})`, so it pins the sole constructor. The set is covered.

Calls made (1 of 6):
1. One `grep` over `internal` for `Topology{`, `.notifier()` and `gcAutoThreshold`. My context held no all-package search of these sites.
