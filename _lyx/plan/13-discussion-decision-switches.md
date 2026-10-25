# Card 13 — discussion-decision-switches

**Edit:**
- `internal/shedadapters#BurlerHalfDeps`
- `internal/shedadapters#BurlerFixProducer`
- `internal/shedadapters#NewBurlerFixProducer`
- `internal/shedadapters#BurlerFixProducer.sendDecisionFindings`
- `internal/shedadapters#TestBurlerFixProducer_Call_SendsDecisionFindingsToTheParent`
- `internal/shedadapters/doc.go`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#burlerFixEntry`
- `internal/shedrecipe#TestBouncerEntry_BlockingEscalatesAndColorSegment`
- `internal/shedrecipe#TestBurlerFixEntry_RoutesDecisionFindingsThroughEnvNotifyParent`
- `contracts/recipes/loom-recipe.yaml`
- `contracts/specs/shed-recipe-spec.md`

**Create:**
- `internal/loomrecipe#TestDecisionSwitches_SetTogetherOnLoomsDiscussionRowsOnly`

**Uses:**
- `internal/shedrecipe#layoutBouncerRound1Report`

**Intent:** D8's two switches, looked up by row config key (`PATTERN-shed-recipe-registry`).

- Bouncer rows: `bouncerEntry` reads the optional boolean `escalate_decisions` (default false) into `BouncerConfig.EscalateDecisions`, and lists it in its `configRejectUnknown` keys and its doc.
- Fix rows: `BurlerHalfDeps` gains `DecisionsEscalated bool`, meaning the segment's Bouncer escalates the round's `decision` findings itself.
  `burlerFixEntry` reads the optional boolean `decisions_escalated` (default false) into it.
  `NewBurlerFixProducer` sets `opts.DecisionsRouted` when `NotifyParent` is set or `DecisionsEscalated` is, so the fixer never gets a `decision` finding, and `sendDecisionFindings` posts nothing when `DecisionsEscalated` is set, so the Bouncer's escalation is the one parent message per round, a deliberate change to `2e265a4fc`'s notice in that segment only.
- `loom-recipe.yaml`: `escalate_decisions: true` on Discussion-Bouncer and `decisions_escalated: true` on Discussion-Burler-Fix, each with a comment pointing at the other; Plan-Review and Code-Review rows, and darn's, set neither and keep today's fix-row notice.
- Tests: `TestBurlerFixProducer_Call_SendsDecisionFindingsToTheParent` gains two rows with `DecisionsEscalated`, one with a notifier and one without, each wanting no posted line and the fix run with `DecisionsRouted` set.
  `TestBouncerEntry_BlockingEscalatesAndColorSegment` observes only the outcome, so its rows overwrite the round-1 review `layoutBouncerRound1Report` writes with one holding a `decision`-class finding: `escalate_decisions: true` wants Awaiting, and an absent key wants the outcome that review settles to today.
  `TestBurlerFixEntry_RoutesDecisionFindingsThroughEnvNotifyParent` gains a row setting both `env.NotifyParent` and `decisions_escalated: true`, wanting zero posted lines and `DecisionsRouted` true on the fix opts.
  `TestDecisionSwitches_SetTogetherOnLoomsDiscussionRowsOnly` parses the embedded loom and darn recipes and asserts both keys are true on exactly Discussion-Bouncer and Discussion-Burler-Fix and on no other row of either recipe, so Plan-Review and Code-Review keep their behaviour.
- Docs: the shedadapters doc's fix-row paragraph and `shed-recipe-spec.md`'s Bouncer key list name both keys.

What it can skip or let through: a fix row so switched sends no notice of its own; the bound is the Bouncer's escalation of the same findings, which the recipe test pins set together with it.
On card 12's handled-false paths, a round carrying an escalation record or a decision file of another cause, which only an older binary or a flipped switch leaves, the round's `decision` findings reach neither the fixer nor a parent notice.
Over an escalation record or a parent-recorded decision of another cause, that cause's escalation brief points the parent at the round's review; over an automatic `budget` decision file no brief is written, so the findings reach the parent only through the round's review in the run records.

**ImpactSummary:** Two optional row keys and one fix-row field; only loom's two Discussion-Review rows set them.
