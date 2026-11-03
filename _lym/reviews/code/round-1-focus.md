---
round: 1
focus:
  - "internal/loomcli/reviewstatus.go, every Waiting hook (gateWaiting, verifyWaiting, shuttleWaiting, parentLockWaiting): does each note's Steady form replace every elapsed rendering in its Text and nothing else, including the branches the diff leaves untouched (the running-verify branch, the shuttle wait, the next-step fall-through, the combined gate-plus-rest note)?"
  - "Is any other time-varying text still in a note's Steady form, such as a holders clause, an attempt counter or a detail string, that would make the watch print on every poll, or any real change (wait kind, detail, producer, holders) that the Steady form now hides?"
  - "internal/shedverbs/status.go statusWatchLine and PrintStatusLinesOnChange: is the change key built so that a row, state, wait-kind or detail change still prints, the unavailable line still dedupes, and a wait starting or ending still reprints? Check the first-poll and hook-error paths."
  - "Callers of the changed signatures (Hooks.Waiting, PrintStatusLinesOnChange, formatWaitNote, checklistPoll): does every caller across the repository compile-consistently use WaitNote, and does the one-shot status and the checklist pane still show the elapsed time via Text?"
  - "Scope against the Goal: the watch ignores the elapsed counter while the one-shot status keeps it. Does the diff change anything else, such as the envelope's `waiting` key, the status file schema, or the human view?"
  - "contracts/specs/loom-status-spec.md, docs/overview.md and the verb help in internal/loomcli/cli.go and internal/shedverbs/status.go: do the new sentences match the behavior the diff implements, and do the repository's rules for doc updates and comment conventions hold for every new or changed comment?"
  - "Tests in reviewstatus_test.go and status_test.go: is each new behavior covered once, with no duplicate of an existing test, and does `steadyOf` risk passing by construction because it derives the expected Steady from the same Text?"
---

Seed focus for round 1.
The record holds the board entry alone under `## Goal`, so the entry is the measuring stick.
