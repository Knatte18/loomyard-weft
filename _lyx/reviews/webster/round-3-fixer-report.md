## Changes

- F1 (BLOCKING, design): fixed as proposed.
  `publishRetryResume` in `internal/battenshed/publishretry.go` now calls `p.noticeDriverBusy(ctx, w, err)`, passing the `Spawn` error as `innerrun.go` does with `spawnErr`.
  `go vet` passes for `./internal/battenshed/` with and without the `integration` tag, and `lyx gate test ./internal/battenshed` is green.

## Disputed

None.

## Deferred

None.
