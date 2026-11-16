# Card 19 — plan-gate-reads-active

**Edit:**
- `internal/loomshed#ValidatePlan`
- `internal/loomshed#WebsterRecordNote`
- `internal/loomshed#TestNewPlanGate`
- `internal/loomshed#TestWebsterRecordNote`
- `internal/loomshed/doc.go`

**Uses:**
- `internal/weaverengine#Active`
- `internal/runplan#DoneCards`
- `internal/runplan#EditedDoneCards`

**Intent:** The plan gate's done-card freeze and the Plan-Review rubric's frozen-card note read the active implementer's state (D13).

- `ValidatePlan` reads the active implementer's `runplan.State` through `weaverengine.Active` over the anchor with a nil `Setting`, and freezes its done cards with `runplan.DoneCards` and `runplan.EditedDoneCards`.
  The setting is never needed here: with neither implementer begun there is no state and no frozen card whichever implementer the setting names, and the seam reads the setting only then (card 6), so the gate keeps its signature and needs no config load.
  A seam refusal is a returned error, never a finding, as an unreadable record is today.
  The seam picks the implementer by begun batches first, so Weaver's done cards stay frozen after `tapestry.implementer` is flipped to `webster`.
- `WebsterRecordNote` renders the active implementer's record the same way, opening with the implementer's name; a seam refusal renders the fixed unknown-frozen-cards note with the refusal logged.
  Its name stays, as its readers pin it.
- `loomshed/doc.go`'s plan-gate paragraph says the run record is the active implementer's.

`TestNewPlanGate` gains rows: a Weaver state with a done batch freezes its card, also with the setting `webster`; both implementers begun is a returned error.
`TestWebsterRecordNote` gains a row rendering Weaver's begun batches.

**ImpactSummary:** Two loomshed readers switch their state source to the seam; Webster-only runs read the same record as before.
