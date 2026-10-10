# Plan-Review round 1 — fixer report

Every finding was validated against the code and the decision record, and every one held; all eleven are fixed.
`lyx loom validate-plan` passes over the fixed plan (`ok: true`, informational `create-new-unit` findings only).

## Fixed

- **F1** (card 11): `internal/shedrecipe#TestNames` joins the `**Edit:**` list, and the Intent says `MultiLLM` joins `ShipsExpectedEntries` in sorted position.
  Confirmed: `internal/shedrecipe/registry_test.go:83` pins the exact `Names()` list.
- **F2** (card 6): an advisor that reaches Done is recorded on its `SeatResult` and queues nothing (D7); only a died, timed-out or wait-errored advisor queues an `advisorNotice` line.
  The Done-advisor row in `engine_test.go` now asserts no notice.
- **F3** (card 6): on a `checkName` mismatch the misnamed seat's strand is stopped through `stopSeat` before the start error is recorded, and a failed stop returns `ErrSeatNotStopped`.
  `engine_test.go` gains the chair and advisor formed-name rows asserting the stop.
- **F4** (cards 3 and 5): card 3's Intent states the advisor-to-chair route's bound per D6: prompt-enforced only, a disobeying advisor can reach any `ListAgents` name, bounded by no advisor prompt naming other sessions, advisors never interactive or notifying, and no delegate authority.
  Card 5's Intent states what naming the chair as an advisor's parent lets through, a chair instruction including relayed advice, and what bounds it: the own-output write rule, the advisor's settings' denies and `Interactive` false.
- **F5** (card 5): the `prompt_test.go` row now pins the refusal `FillWith` makes, a seat stencil declaring a marker absent from its value map, refused with the marker named.
  Confirmed: `internal/stencil/stencil.go:39` checks declared markers against values only.
  The optional `Table.Validate` check for `parent_directive`/`edit_directive` was not added: the decision record's Constraint puts rendering both markers on the consumer's stencil, and no consumer stencil ships in this task.
- **F6** (card 3): the chair block's planned content now says the chair asks an advisor by `SendMessage` to its strand name from `advisor_names`.
- **F7** (card 6): a chair `Wait` error stops the chair and every live advisor, as burler's `decideRound` does (`internal/burlerengine/handoff.go:203`), and returns the wait error or `ErrSeatNotStopped`; `engine_test.go` gains that row.
- **F8** (card 11): the unknown-row-key row is dropped from `entries_multillm_test.go`; the Intent names `TestEveryEntry_RejectsUnknownConfigKey` as its cover through the new `minimalEntries` row.
- **F9** (card 12): the nested-include assertion is dropped from `multillm_fixture_test.go`.
- **F10** (card 5): the two `contracts/stencils#SeatDirective*` uses are replaced by `internal/testkit/stencilkit#Seed`; the test's seeded directory is named as `stencilkit.Seed`'s, which seeds card 3's registered blocks (`internal/testkit/stencilkit/stencilkit.go:28`).
- **F11** (cards 4 and 5): `ReservedMarkers` moves to card 4's `**Create:**` with the list's declaration and is pinned in card 4's accessor test; card 5 drops `Table.Validate` from `**Edit:**`, lists `ReservedMarkers` under `**Uses:**`, and its `ImpactSummary` is updated.

## Disputed

None.

## Deferred

None.
