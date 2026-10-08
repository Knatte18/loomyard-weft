# Webster round 2 — fixer report

## Fixed

- **F1** (`935d3a91b`): the CrossClientControl comment in `internal/reedcli/smoke_dotfill_test.go` now states the reviewer's measured split of the 12-row toucher pane (two status rows, three title rows, seven content rows, about two of them the first strand's).
  It drops the false claim that the second strand's output does not fit, and gives the reason for waiting on the first marker alone: its rows show the marker, which proves the attach has rendered.
  `go test -tags tmux ./internal/reedcli/ -run TestSmokeDotFill` passes; `lyx loom lint-comments` is clean.
- **F2** (`f88027d26`): `SetWaitMark`'s doc in `internal/reedengine/io.go` now says the status bar renders the mark on the strand's button, matching `doc.go`.
  `go vet` and `go test ./internal/reedengine/` pass.

## Disputed

None.

## Deferred

None.
