# Card 2 — Seats carry a ModelChoice

**Edit:**
- `internal/seatengine#Seat`
- `internal/seatengine#seatSpec`
- `internal/seatengine#promptTable`
- `internal/seatengine#TestSeatPrompts_ValuesPromptsAndSpecs`
- `internal/seatengine/doc.go`
- `internal/burlerengine#Profile.reviewTable`
- `internal/burlerengine#Profile.fixTable`
- `internal/loomengine#resolveModelChoice` -> `func resolveModelChoice(raw string, reg modelspec.Registry) (shuttleengine.ModelChoice, error)`
- `internal/loomengine#DiscussionTable`
- `internal/loomengine#PlanTable`
- `internal/loomengine#ResolveRework` -> `func ResolveRework(cfg Config, reg modelspec.Registry) (shuttleengine.ModelChoice, error)`
- `internal/loomengine#ReworkTable`
- `internal/loomcli#loomCLI.wire`
- `internal/darnengine#DarnTable`
- `internal/landingshed#describeSessionTable`
- `internal/shedrecipe#resolveSeatModel` -> `func resolveSeatModel(entry string, i int, spec string, reg modelspec.Registry) (shuttleengine.ModelChoice, error)`
- `internal/shedrecipe#multiLLMSeat`
- `internal/loomengine#TestDiscussionTable`
- `internal/loomengine#TestPlanTable`
- `internal/loomengine#TestReworkTable`
- `internal/loomengine#TestResolveReviewAndJudge_MalformedSpec`
- `internal/darnengine#TestDarnSpec`
- `internal/landingshed#TestDescribeSpec_ComposesSpec`
- `internal/shedrecipe#TestMultiLLMEntry_ComposedTable`
- `internal/shedrecipe_test#TestMultiLLMFixtureRecipe_BuildsAndRunsAgainstFakeSeats`

**Intent:**
D3 for `seatengine.Seat` and the D2 mapping for every resolver that feeds a seat.
`Seat` replaces its `Model`, `Effort` and `Version` fields with an embedded `shuttleengine.ModelChoice`, so reads such as `seat.Model` keep working, and `seatSpec` sets `ModelChoice: seat.ModelChoice`, so a seat spawns on its choice's engine.

The seat resolvers return a whole choice built by `shuttleengine.NewModelChoice`:
`resolveModelChoice` (Discussion-Write's lead and advisors, Plan-Write's lead), `ResolveRework` (PR-Rework's lead; `loomCLI.wire` still only checks its error), `resolveSeatModel` (the MultiLLM recipe entry's seats), and the inline resolutions in `DarnTable` and `describeSessionTable`.
Each table builder sets its seats' `ModelChoice` from that value.
`burlerengine`'s `reviewTable` and `fixTable` build their seat's choice from `RunOpts`' fields as today (card 3 hands them a whole choice).
`seatengine/doc.go` says a seat carries its model choice, engine included.

Tests, each a row or assertion in the builder's existing test:
`promptTable` sets the chair's choice with `Engine: "claude"` and `TestSeatPrompts_ValuesPromptsAndSpecs` asserts the chair spec's `Engine` beside its model triple;
`TestDiscussionTable` asserts the lead's and each advisor's `Engine` is the resolved alias's engine (`claude`) (the escape form is card 1's `TestNewModelChoice` row);
`TestPlanTable`, `TestReworkTable` (both the `rework` and the fallen-back `plan` key rows), `TestDarnSpec`, `TestDescribeSpec_ComposesSpec` and `TestMultiLLMEntry_ComposedTable` assert the lead seat's `Engine` is `claude`.
`TestMultiLLMEntry_ComposedTable`'s and `TestMultiLLMFixtureRecipe_BuildsAndRunsAgainstFakeSeats`' seat literals move their model fields into `ModelChoice`.

**ImpactSummary:** one struct field set becomes an embedded choice; the six seat-table builders and their tests change, every other seat read compiles unchanged through promotion.

**Commit:** 2: Seats carry a ModelChoice built by NewModelChoice
