---
round: 4
focus:
  - "Halts with no history entry: a producer that returns an error (agent never started, agent died before its verdict) leaves the run `failed`, and `appendHistory` in `internal/shedengine/run.go` appends nothing for an empty outcome. In `_lym/discussion/decision-record.md`, check D3's stop count (including hand-back and arbiter restep records, and a stop the tackler escalates and the arbiter resumes) against such stops: does each count exactly once, across a driver restart too, so the third halt at a row can be reached?"
  - "Recording and keying such halts: check where D7 puts `class:`, `settle:`, `halts_at_row:` and `halt:` when the halt has no history entry, whether two successive failed halts at one row get different keys, and whether D8's dedup, D7's posted record and D10's chase follow from that."
  - "Acceptance criteria and the declared `PATTERN-stop-settlement` text: do they agree with the decisions on entry-less halts, with a named test intent and tier for the count and for the record and post of a second such halt?"
---
