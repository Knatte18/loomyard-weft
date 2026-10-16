# Card 10 — review-loom-declared-changes

**Edit:**
- `internal/burlerengine#Geometry`
- `internal/burlerengine#Engine.Review`
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireLightweight`
- `internal/loomengine#DiscussionTable`
- `internal/loomengine#PlanTable`
- `internal/loomengine#ReworkTable`
- `internal/burlerengine#TestEngine_Review_TableShape`
- `internal/burlerengine#TestEngine_Review_Outcomes`
- `internal/shedadapters#TestBouncer_JudgeCall_PatternDirective`
- `internal/shedadapters#TestBouncer_JudgeCall_PatternDirectiveFailureDegrades`
- `internal/loomcli#TestWireLightweight_FillsThePathsWithoutLoadingAnyConfig`
- `internal/loomengine#TestDiscussionTable`
- `internal/loomengine#TestDiscussionTable_Refuses`
- `internal/loomengine#TestPlanTable`
- `internal/loomengine#TestReworkTable`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `plan:internal/discussionparser#DeclaredPatternChanges`
- `internal/shedadapters#Bouncer.degrade`

**Intent:**
D5.4 for every other `pattern.Directive` caller: the Burler reviewer (RoleReviewFix), the Bouncer judge (RoleJudge), the Discussion-Write designer (RoleDesigner) and the Plan-Write and PR-Rework writers (RoleImplementer) render the declared changes, each reading the record through `discussionparser.DeclaredPatternChanges` at a path it is told.
`D5.7`'s landing check at Webster-Review reads the declarations rendered into its directive here, so a bullet `decision add` appended after the plan gate reaches every later directive.
A darn run takes no PATTERN declaration (D5.4), so its review rows are told no record.

- `shedrecipe.Env` gains `PatternRecordPath`, the record whose declared PATTERN changes the review rows render: `loomCLI.wireLightweight` and `loomCLI.wire` set it to `loomengine.DiscussionDecisionRecord(location)` on a loom run and leave it empty on a darn run, beside the `DecisionRecordPath` they already set, which the darn writer and the escalation brief keep reading.
- `burlerengine.Geometry` gains `PatternRecordPath`, empty for none; `loomCLI.wire` sets the burler geometry's field from `c.env.PatternRecordPath` after `hubgeom.BurlerGeometry` builds it, so one value decides both review rows, and `standalonegeom.BurlerGeometry` leaves it empty, so a standalone burler round renders none.
  `Engine.Review` reads it before composing the prompt; an error returns `burler: <err> re-step the review row with "lyx loom start"` with no reviewer spawned.
  The review segment's Burler-round row already returns any `Review` error to the driver unchanged, which halts the run, so the refusal is reached through `Review` and that row gains no code.
- `shedadapters.BouncerConfig` gains `PatternRecordPath`, which `shedrecipe.bouncerEntry` fills from `env.PatternRecordPath`; `Bouncer.judgeCall` reads it beside its `pattern.Directive` call, and an error degrades the judge call through `degrade`, its message carrying the error and "then run `lyx loom start` to re-step the row", with no judge spawned.
- `DiscussionTable`, `PlanTable` and `ReworkTable` read `DiscussionDecisionRecord(layout)`; an error fails the table as `loom: DiscussionTable: <err> re-step the row with "lyx loom start"` (and `PlanTable` and `ReworkTable` alike).
  On a first Discussion-Write no record exists yet, which renders no changes.
  `darnengine.DarnTable` keeps nil as card 8 left it.
- `refusal-spec.md`: `## shed` gains, beside the review segment's other Bouncer rows, a row for the Bouncer's judge call and one for the Burler-round row's review, each triggered by a malformed `## PATTERN changes` section, a correctness halt whose way forward is the operator's record fix, `lyx loom validate-discussion`, then `lyx loom start`; `## loom` gains the same row for the Discussion-Write, Plan-Write and PR-Rework tables.

Tests:
`TestEngine_Review_TableShape` gains a row whose geometry's `PatternRecordPath` record declares one bullet: the reviewer's prompt directive holds it.
`TestEngine_Review_Outcomes` gains a row whose record has a malformed section: `Review` returns an error naming `lyx loom validate-discussion` and no seat runs.
`TestBouncer_JudgeCall_PatternDirective` gains a column and a row whose fixture config's `PatternRecordPath` holds one bullet, which the judge prompt carries after the overview.
`TestBouncer_JudgeCall_PatternDirectiveFailureDegrades` becomes table-driven over its current case and a malformed-record case, each degrading with no judge spawned and its own log message.
`TestWireLightweight_FillsThePathsWithoutLoadingAnyConfig` asserts `c.env.PatternRecordPath` equals `loomengine.DiscussionDecisionRecord` of the location on the loom wiring and is empty on its darn wiring.
`TestDiscussionTable`, `TestPlanTable` and `TestReworkTable` gain a row whose record declares one bullet, which the seat's `pattern_directive` value carries; `TestDiscussionTable_Refuses`, `TestPlanTable` and `TestReworkTable` gain a row whose record's section is malformed, which fails the table naming `lyx loom validate-discussion`.

**ImpactSummary:** five callers read one more file before their spawn; every loom path is told the record by its env, geometry or layout, and standalone and darn paths render none.
