# reed: stray strands after a crash

`reed-crash-strands` · Note · bug, reed

Two reed bugs seen on 2026-10-06 when the machine crashed and the runs were brought back.

## Bare claude strand above a dormant orch (issue #408)

After a machine crash, opening the hub prime in VS Code left an empty `ly:claude` strand in the prime's reed session, with the `ly:orch` strand stacked below it.

Cause: the folder-open task chain written by `internal/vscode/config.go` runs `lyx reed add --if-absent --unless-name orch --name claude`.
`--unless-name` skips the add only while the orch strand is live (`AddStrandUnless`, `internal/reedengine/strand.go`).
After a reboot the orch strand is persisted but dormant, so the check passes, a bare `claude` strand is added, and the orch is resumed into the same session afterwards.

Expected: a prime whose reed state holds an orch strand, live or dormant, gets no bare `claude` strand on folder-open.
Possible fixes: let `--unless-name` also match a persisted dormant strand, or have the folder-open chain run `lyx reed resume` before the add.

## Resume revives strands whose work is done (issue #409)

After a machine crash, `lyx reed resume` in the `test-suite-prune-core` pair recreated two strands: the driver and `ly:test-suite-prune-core:burler`.
The burler's round had finished before the crash (verdict written and committed), and the run had moved on.
When the driver re-ran its shed step, the next producer spawned its own strand (`describe`), so the pair showed a dead-weight burler pane beside the live one.

Cause: a strand stays persisted in reed's state after the work it was spawned for is done, since only the shuttle wait that would tear it down died with the crash; `resume` replays every persisted, non-live strand without asking whether its work is still pending.

Expected: after a crash, `reed resume` brings back only strands with work left, or a strand is removed from reed's state as soon as its shuttle run reaches a final outcome, independent of the waiting process.
