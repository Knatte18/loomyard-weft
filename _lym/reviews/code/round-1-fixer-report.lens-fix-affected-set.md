```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Finding F1 is the only one whose premise is a set: the git-backed rows of `TestUnwire_LeavesSiblingWorktreeUndirtied` that need a stripped-line assertion.

- **Rows:** the fix covers every row that has a line to strip. Only `UnwiredPairAloneAtItsAnchor` has one: it unwires at `sub` while the kept pair sits at `.`, so `/sub/_lym` is stripped.
- **Rows with an empty `wantStripped`:** `PairsAtTheHubAnchor` and `KeptPairAtTheSameAnchor` share the anchored path with the kept pair, so by `excludeLinesToStrip` their anchored line is kept. The bare `_lym` line is also kept, because a sibling still wires it.
- **Other call sites of `unseedGitExclude`:** it is also called from `junction.go:410`, `reconcile.go:878` and `anchorchange.go:493`. The finding scopes itself to the `Unwire` call site (`unwire.go:84`), which this test drives, so those sites are out of scope.

Tool calls:
1. `grep -rn "unseedGitExclude(" internal --include=*.go`: to list the call sites of the finding's function, which my context lacked.
