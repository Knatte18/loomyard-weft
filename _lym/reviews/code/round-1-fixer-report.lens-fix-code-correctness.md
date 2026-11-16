```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

- Each substitution swaps an untyped string constant for the identical literal it replaced: `"on_land"` → `onLandKey` (`front.go`, `replay.go`, `store.go`, `render.go`) and `"pull_request"` → `pullRequestKey` (`replay.go`, `store.go`, `render.go`). The values match the struct tags at `task.go:49` and `task.go:67`, so behaviour is unchanged.
- `runFieldLines` with `PullRequest="merge-in"` still gives `"pull_request: merge-in"`, and with `OnLand="x"` it gives `"on_land: x"`. The golden bytes are unchanged.
- `checkFrontPayload`, `FileFrontTask` and `UpsertPayloadKeys` see the same string values, so every caller holds. The signatures didn't change.
- The new constants are unexported, with no name collision: `go build`, `go vet` and `lym gate test ./internal/boardengine` all passed (852 passed, 0 failed).
- The review's F1 `fix` text also named `front.go:57`. That line is covered, and the `"front"` literal beside it is outside F1.
- No test was added. F1 and F2 are not coverage gaps, so none was required. The existing golden row that pins `pull_request:` and `on_land:` would still catch a wrong constant value.

Tool calls:
1. `git diff HEAD~2 --stat` plus the changed lines: my context held the pre-commit edits but not the committed diff, so I confirmed no unintended change.
