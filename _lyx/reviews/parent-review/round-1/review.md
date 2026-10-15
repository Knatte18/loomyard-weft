---
verdict: APPROVED
---

## Scope against the board entry

Every Scope point of `engine-usage-readers` is covered by the decision record:

- The neutral per-message usage record and the transcript locator in `shuttleengine`, implemented by `claudeengine`: D1, D2, D3.
- tokencount reads through each run's engine and deletes its own parser: D1, D2, D6.
  D1 reads "each run's engine" per session, as the union of the engines that recorded sessions under the run's worktree; this follows from `engine-model-choice` letting one run's roles resolve to different engines, and fits the goal.
- Prices move into each engine behind a capability: D4.
- Calibration reads the engine's instruction files: D8, through the existing `InstructionFileLister` capability in `internal/shuttleengine/engine.go`.
- The "Seen" duplicate of the Claude usage parse in `count.go` and `claudeengine/usage.go` ends in one shared parse: D5.
- The entry's Out, `report-timing`, stays out.

The one addition beyond the entry, D7's header line naming a registered engine without the locator, is declared in Scope and prints only when such an engine exists.
It is the reporting side of making the locator an optional capability, so it is inside the goal of reading through each engine rather than separate work.
The deleted `-projects` flag and the offline read set in `shuttleengines` (D6) are consequences of removing the Claude path from tokencount, not new work.

No other board entry is duplicated or contradicted.
`codex-engine` is left to implement the locator, reader and pricing for its own sessions, which the record names as an open risk rather than taking on.
`session-sync` touches Claude transcripts for a different purpose and has no design.

## Overlap with peers and parent commits

`engine-model-choice` (row 12; this run's own row 3):

- Instruction files: its D7 makes the live `websterengine.MerriamBase` take the names from Master's engine; this run's D8 makes calibration's reconstruction take them from the Webster session's engine, and puts `MerriamBase` explicitly Out.
  The two decisions mirror each other and do not conflict.
- `round-N-usage.yaml`: its D4 adds an `engine` key from `Result.Engine`; this run's D5 keeps `SessionUsage`'s readings unchanged while sharing its parse with the new reader.
  Different behaviour at different places; this run's Open risks already asks whichever lands second to re-check the readings.
- `internal/shuttleengine/doc.go`, `internal/shuttleengine/engine.go` and `claudeengine` are changed by both at different places: a text-only overlap.
- PATTERN: the peer adds `PATTERN-model-choice-mapping`; this run adds no PATTERN entry.
- Shared bookkeeping: `cmd/lyx/testdata/test-budget.yaml`.
- Landing order: `engine-model-choice` is further along (row 12 against row 3), so it lands first and this run takes its changes at Merge-In.

Parent commit `a44ef5e Creel stage 1 audit report` adds only `docs/research/creel-audit-2026-10-15/report.md`; no overlap.
