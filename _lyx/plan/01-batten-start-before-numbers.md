# Card 1 — batten-start-before-numbers

**Create:**
- `docs/research/batten-start-2026-10-18/README.md`

**Intent:** Record D2's before numbers, on the code as it stands before any later card of this plan, as a measurement report.

Measure a batten start from `Worktree-Create`'s entry to `Run-Shed`'s entry, per row and inside each row: fabric add and its origin probes, the status write, the no-op marker, the merge probe, the status commit, the status push, the config reads and the row's own action.
The numbers come from the logger's trace timestamps and the run's status history only, with no new timing infrastructure.
The per-row numbers come from the status history's transition timestamps of `TestBattenIntegration_Rows`' rows, run through `lyx gate test --tags integration ./internal/battencli`.
The inside-a-row breakdown comes from the trace of a live `lyx batten run` start, at `-vv`, since root's pre-run arms the trace sink only outside a test binary (`cmd/lyx#newRoot`); take one when the machine allows it.
The report names the rows it measured live and the rows it measured only in the integration-tier drive, the command it ran, and where each number came from.
It ends with a section headed `After`, left to name the later card that fills it, so card 16 adds its numbers beside these.

A measurement report records one run at a point in time, so its tallies and timings are the document's content (scribe:prose's measurement-report exception).

**Commit:** 1: Research: batten start before numbers
