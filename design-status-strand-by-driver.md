Operator decision: a run's tmux session shows exactly one driving surface, chosen by the run's driver.
- `llm` driver: only the ly-drive session pane; no loom-status strand (the driver already reports progress, the status band above it is noise).
- `go` driver: only the loom-status strand.

Today `lyx loom start` always ensures the status strand (`internal/loomcli/arm.go` calls `ensureStatusStrand`, `internal/loomcli/sharedbootstrap.go`), and an llm-driven run adds the ly-drive strand next to it.
Wanted: `ensureStatusStrand` runs only for a `go`-driven run (driver read from the run's seed), and a resume of an llm run whose session still holds a status strand from an older build removes it.
`lyx loom status` stays available as a command either way.

Driver pane height: under the `083e992cf` layout rule every strand but the bottom-most collapses to `collapsed_rows` (3), so the ly-drive pane shrinks to 3 rows as soon as an agent strand spawns below it and becomes unreadable (operator report).
Wanted: the driving surface keeps a readable height of its own (a `reed.yaml` key such as `driver_rows`, default ~16) while other non-bottom strands keep `collapsed_rows`; the agent below takes the rest.