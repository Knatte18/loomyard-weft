# Orch color retyped after every deploy

`orch-color-on-reexec` · Note · bug, orch

## Seen

The orch pane gets `/color blue` typed long after a distill, often minutes later and mid-work.
An orch agent traced it on 2026-10-15: the reload color itself lands 0.5 to 1.2 s after each compaction ends (once held 2m29s by the operator typing in the pane, which is the intended guard).
The late colors are extra ones: every deploy changes the `lyx` binary, `selfexec.Check` re-execs the watcher in place (`internal/selfexec/selfexec.go`), and the fresh process's first tick runs `initCursor` in `internal/orchengine/watcher.go`, which sets `colorPending = true` unconditionally.
`typeStartColor` then types the color on the first idle tick, so it arrives once the orch's running turn ends.
17 such colors were typed from 2026-10-14 18:36Z to 2026-10-15 11:05Z, each one on a session already colored.
`typeStartColor` writes no typed-injection log line, unlike the reload step, so these never show in the watcher log.

## Idea

Re-type the color only when something can have reset it: persist in the watcher `State` the strand and session start the color was last typed for, and set `colorPending` in `initCursor` only when they differ.
A re-exec on the same session then types nothing; a new session still gets its color.
Log the start color like the reload steps do.
