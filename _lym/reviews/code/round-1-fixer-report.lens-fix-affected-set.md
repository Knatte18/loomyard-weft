```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

The search that produced the finding now matches no boardengine code literal for either key. The remaining matches are prose, or sit in other packages.

- **boardengine.** `render.go`, `replay.go`, `front.go` and `store.go:500` all use `pullRequestKey` and `onLandKey`. The only literals left are the struct tags, which Go requires, and comment prose (`store.go:494`, `front.go:52`, `doc.go:366`).
- **Outside boardengine, F1 covers no more.**
  - `internal/beamcli/mergeparent.go:79` is a payload map key in another package. The review scoped F1 to boardengine, and the constants are unexported, so it can't use them.
  - `internal/dobbyengine/config.go:43` and `:63` are beam.yaml keys, a different file.
  - The `landingshed` error strings and the `boardcli/cli.go` help text are message text, not key uses.
- **F2.** `doc.go` no longer has an "entry cell" sentence about profile; its only remaining entry-cell paragraph is the rewritten one at `:366`.

Tool calls made:
1. One grep over all non-test Go files for both key literals and the `pull_request: ` / `on_land: ` line prefixes, plus a grep of `doc.go` for "entry cell". My context held only the files the fixer read, so it couldn't show other packages or the final state of these sites.
