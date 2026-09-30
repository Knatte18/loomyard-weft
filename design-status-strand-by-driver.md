Operator decision: a run's tmux session shows exactly one driving surface, chosen by the run's driver.
- `llm` driver: only the ly-drive session pane; no loom-status strand (the driver already reports progress, the status band above it is noise).
- `go` driver: only the loom-status strand.

Today `lyx loom start` always ensures the status strand (`internal/loomcli/arm.go` calls `ensureStatusStrand`, `internal/loomcli/sharedbootstrap.go`), and an llm-driven run adds the ly-drive strand next to it.
Wanted: `ensureStatusStrand` runs only for a `go`-driven run (driver read from the run's seed), and a resume of an llm run whose session still holds a status strand from an older build removes it.
`lyx loom status` stays available as a command either way.
Driver pane height needs no new key: with the status strand gone, the stack is ly-drive over the agent, so `collapsed_rows` is the ly-drive height (hub config set to 16 in weft `main-weft`).