# Dobby log: changes and a heartbeat only, and bounded

`dobby-log-bounded` · Note · enhancement, workspaces

## Seen (OSL-1046, 2026-10-26)

The prime's `dobby.log` grew by 1423 lines between 09:02 and 17:00, nearly all of them `pass <slug> continue`, one per pass of every run in flight, about every 30 seconds; one run alone wrote 745.
Nothing bounds the file: `SpawnDetached` (`internal/dobbyengine/start.go`) appends the detached dobby's stdout and stderr to it, and `logLine` (`internal/dobbyengine/passes.go`) writes the pass lines there, outside `internal/logger`, whose trace files a retention sweep bounds by age and count.

## Idea

- Dobby logs through `internal/logger`, so its log falls under the same retention; or, if the log keeps its fixed place beside `state.json`, it gets a size bound of its own, rotated to one previous file.
- Only changes are logged: a dobby start or re-exec, a run started, halted, landed or failed, a pass that fails, and a board-tidy spawn or end.
- A heartbeat line every 10 minutes names the runs it drives and their states, so the log shows within ten minutes when a dobby died.
- It runs as its own task after the rename to `lym`, never inside it, since the rename is a mechanical change that runs alone (operator, 2026-10-26).
