---
round: 6
focus:
  - "Readers of the events file's mtime or size: `internal/shuttleengine/activity.go` `readRunActivity` takes the newer of the transcript mtime and the events file mtime as `LastActivity`, and batten's quiet notice compares it with a threshold. Does the decision record's D1 Bound name every such reader, and state what the new hook and stamp lines do to each one in shadow mode, with a bound or a mechanism that keeps them out of the reading?"
  - "A signal parser fed incrementally: `readEventsFrom` consumes every complete line past its offset, and a stamp line is complete on its own. Does the decision record say where a pending stamp lives when a poll falls between a stamp line and its payload line, so that `since` is the same for the wait loop, the orch watcher and a whole-file reader, and does an acceptance row split a batch at that point?"
  - "History entries D3 calls signal-only: `outstandingBackgroundTasks` in `claudeengine/background.go` uses the Stop payload's `background_tasks` and otherwise reads the current transcript at parse time. Does D3 limit the signal-only claim to entries that really rest on signals alone, and does it classify the newest turn start, which reads `idle-stalled` or `busy` depending on whether the reader marks an interrupt, consistently with that claim?"
  - "Precedence versus the output-files-first statements: do the Constraints line on `PATTERN-completion-signal`, the D3 `idle-done` paragraph, the precedence rules and the acceptance rows agree on whether `dead` and `unknown` rank above a turn end with every output present, and does the record say which states `PATTERN-completion-signal` governs?"
---

Entries restate the sites they depend on and make no claim about the answer.
None targets a `Do not flag` item: they ask about stated bounds on new appends, reader consistency and ordering, not about missing alternatives or enumerations.
