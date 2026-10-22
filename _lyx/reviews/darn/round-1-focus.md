---
round: 1
focus:
  - "internal/shedverbs/loopwait.go and looppid.go: does the dead-loop stop name the in-flight producer and trace id, and pick the in-flight step's first trace file as trace_file, with the loop-log fallback only when no trace id or no file exists? Is the trace_id written to the pid record at child start on every path, and read safely when the record is stale or absent?"
  - "internal/loomengine/anomaly.go and internal/loomcli/halt.go, looparm.go: does the interrupted flag reach ClassifyHalt from every interrupt-hook caller, and does it leave every blocked kind and the unflagged failed case unchanged? Does any other ClassifyHalt caller or anomaly-kind consumer (docs, switch statements, enumerations of kinds) miss the new interrupted-stop kind?"
  - "Board entry asks for a distinct anomaly kind for interrupt/stalled-session exits and the cause plus in-flight trace file in the halt note's reason: does the diff make both, and does it change anything beyond that, such as the panicking-child reclassification the summary claims?"
  - "Docs and summary claims: do internal/shedverbs/doc.go and internal/loomcli/doc.go match the code, and does every claim in _lyx/landing/summary.md (kept bare loop-exited token in the envelope error, test coverage list) hold against the diff?"
  - "Test economy: do the new loopwait_test.go cases and anomaly_test.go rows duplicate existing coverage, and is any added behavior left without a test?"
---

Seed focus from the board entry (issue #801) and the diff stat; no prior round exists.
