# Retire the loom-status strand; the driver pane is the run status

`retire-loom-status-strand` · Note · enhancement, loom, reed

## Seen

Every run still opens a `loom-status` strand (`statusStrandDisplayName` in `internal/loomcli/bootstrap.go`) at the top of its tmux window, as `quick-recipe` showed on 2026-10-10.
`driver-context` (f5b5218f7) kept it for both driver kinds.
The operator reads a run's state from its driver pane, so the status pane is outdated and only takes window space.

## Idea

Stop spawning the `loom-status` strand; the driver pane is the run's status.
