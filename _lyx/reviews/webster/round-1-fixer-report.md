# Webster round 1 — fixer report

Every finding was validated against the code and fixed; none disputed, none deferred.

## Fixed

- **F1** (`5daf9d89a`) — Reproduced: `TestSmokeDotFill/CrossClientControl` timed out under `-tags tmux` with the toucher pane showing only the three title rows, one prompt line and the two bar lines.
  The toucher pane in `internal/reedcli/smoke_dotfill_test.go` now gets about 12 rows and the observed pane about 27, so the first strand keeps content rows under the two-line bar and the title rows while the toucher stays distinctly smaller.
  The sizing comment states the new arithmetic.
  `TestSmokeDotFill` passes three runs in a row (`-count=3`), so the control still reproduces the dot-fill artifact.
- **F2** (`7d0b13ba0`, `5536b4721`) — Reproduced against `internal/shuttleengine/run.go`: the `/color` line is typed after startup and before the skill load whenever the strand has a color and the spec is not `ColorByCaller`.
  The stub driver script in `internal/loomcli/integration_driverbootstrap_test.go` now skips a leading `/color` line before reading the load message, and its doc comment describes that line.
  The second commit joins the new doc-comment line so `lyx loom lint-comments` passes.
  `TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver` passes under `-tags tmux`.
- **F3** (`bd9aef936`) — `ensureServerAndSessionLocked`'s doc in `internal/reedengine/lifecycle.go` no longer lists the status-line template among the boot validations.
  The comment on the healthy already-up branch is removed, since the code alone says it returns.
- **F4** (`8db276e58`) — The `internal/burlercli/wiring_test.go` header no longer lists `RepoName`, and says `TestReedGeometry` asserts the geometry's fields rather than giving a field count.
- **F5** (`97e8d0883`) — `internal/reedengine/doc.go` now says that launch sets the pane options and boot marks the strand window, and that every attach pre-flight marks the window and re-asserts both pane options on every strand bound to a live pane, matching `attach.go` and `paneoptions.go`.

## Verification

`go build ./...`, `go vet` under `integration`, `tmux` and `llm`, `go test ./...` and `go test -tags integration ./...` pass.
`go test -tags tmux` passes for `internal/reedcli`, `internal/loomcli`, `internal/reedengine` and `internal/burlercli`.
`lyx loom lint-comments --base b033d840a` passes.

## Disputed

None.

## Deferred

None.
