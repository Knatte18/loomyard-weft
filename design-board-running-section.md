# Board README: a Running section before Layer A

`board-running-section` · Note · enhancement, board

The board README gets a `Running` section under Tasks, before `Layer A`, holding every task a run holds: the entries `board-lock-running` locks against scope changes.
A running task appears there and in no layer, so the layers show only work that can still be started or reshaped.

## Shape

- In `renderReadme` (`internal/boardengine/render.go`), split the open tasks on `IsRunStatus` (from `board-lock-running`, `internal/boardengine/runlock.go`) before the layer loop and write them first under `### Running`.
- Use the same predicate as the lock, so the README and the refusal can never disagree on which entries are locked.
- After and Before still render for a running entry.

Waits for `board-lock-running` to land, since `IsRunStatus` comes with it; then the orch fixes it directly on `main`.
