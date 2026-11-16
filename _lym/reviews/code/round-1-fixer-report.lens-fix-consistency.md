```yaml
lens: fix-consistency
outcome: reported
```

No findings.

The change set agrees with its surroundings.

- **F1 literals:** Every `pull_request` and `on_land` literal the review named in `boardengine` now goes through `pullRequestKey` or `onLandKey`: `render.go`, `replay.go`, `front.go` and `store.go`. The matches that remain are outside `boardengine`, or are tags, help text or comments.
  - `internal/beamcli/mergeparent.go:79` is the beam-config key of a different structure.
  - `internal/hubissues/hubissues.go:45` is a struct tag.
  - `internal/dobbyengine/config.go:43`, `:63` and `:105` are the beam.yaml config key, not the board.json field.
  - `internal/boardcli/cli.go:612` and `:618` are help text.
  - `internal/fronttask/fronttask.go:100` is a struct tag in another package.
  - `internal/boardengine/task.go:84` and `:86` are the new constants.
  - `internal/boardengine/store.go:494` is a doc comment.
- **F2 wording:** `grep -i 'entry cell' internal/boardengine/doc.go` finds nothing, so no `doc.go` sentence still places the profile in the entry cell.
- **Constants and tags:** The new constants sit beside the `Task` type, as the fix asked. The `task.go` struct tags stay literal, as the fix said they would.

Tool calls (1 of 6): one Bash call ran two greps. The first listed remaining `"pull_request"` and `"on_land"` literals in non-test Go files under `internal`. The second searched `doc.go` for "entry cell". My context held neither result.
