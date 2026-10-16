# Lens review

Every fork returned a report.

- forced-edits, BLOCKING, fixed differently: proposed re-opening the gate-quiescence decision instead of planning the twill fixer; D7 is that re-opening, and `internal/loomrecipe/gatequiescence_test.go:19-23` already rests on the Stop-payload wait, which the gated Plan-Write lead relies on when it forks under `plan_lenses` (`internal/shedrecipe/entries_planwrite.go:73-83`). Card 7 now edits `gateQuiescenceFailureMessage`, `gatedRowFanViolations` (re-signed to list fans) and `TestNoGatedRowAuthorizesForkSubagents` to state that premise and the fixer audit's backstop, card 3 restates it and fixes `fixTable`'s "never forks" doc, and the plan verify runs `./internal/loomrecipe`.
- row-reachability, BLOCKING, fixed as proposed: card 2's `fork_reading` row in `TestTemplate_OptionalDirectives` gets a per-row placement (after `## Tool-use rules`), and the explore `optional` slice gains `fork_reading`.
- forced-edits, BLOCKING, fixed as proposed: card 8 edits `internal/loomengine#templateConfig`, the test `Config` literal holding `FanReview`.
- criteria-coverage, MEDIUM, fixed as proposed: card 3's engine refusal now takes its step per caller, D7's `set engine: in shuttle.yaml ... or remove <fan key> from the profile, and re-run` for `lyx burler run` and the resume step for a shed row; a row asserts each.
- test-economy, MEDIUM, fixed as proposed: card 6 drops its new top-level test and adds `ClusterFans` rows to `TestBouncer_ClusterExcludesReachesSeedAndJudgePrompts`, listing its helpers under Uses.
- test-economy, MEDIUM, fixed as proposed: card 5 drops `TestBurlerFixProducer_Call_RecordedFans`; its refusal rows join `TestBurlerFixProducer_Call_RefusesARoundWithNoAcceptedReview`, its fan-reaches-FixFan and non-RequiresFans rows join `TestBurlerFixProducer_Call_FinishedFixIsRecorded`, and the re-step row joins `TestBurlerReviewProducer_Call_RecordsRoundFans`.
- test-economy, MEDIUM, fixed as proposed: card 5's resume row moves to `TestBurlerReviewProducer_Call_ProbeAndResume` and its per-round pick rows to `TestBurlerReviewProducer_Call_ProfileAndOpts`; `RecordsRoundFans` keeps only the pre-spawn write and its failure.
- row-reachability, MEDIUM, fixed as proposed: card 5 adds `GotProbeReviewProfiles` and `GotResumeReviewProfiles` to `internal/testkit/shedfake#BurlerHalfRunner` and lists the fake; the resume row asserts the fan through them; the verify runs `./internal/testkit/shedfake`.
- row-reachability, MEDIUM, fixed as proposed: card 7's `TestBouncerEntry_ClusterExcludes` helper writes a hand-built round-1 usage record with `fans_recorded`, `fan` and a review half.
- branch-coverage, MEDIUM, fixed as proposed: card 6 adds a `[""]` row over a recorded empty fan, which reaches the recorded-empty guard.
- branch-coverage, MEDIUM, fixed as proposed: card 5 adds a `ProfileAndOpts` row whose flagless same-fan record drops the exclusion, reaching `lensesRunUnderFan`'s flag check.
- branch-coverage, MEDIUM, fixed as proposed: card 8 adds a mapping-valued `discussion_fix_fan` row reaching `fanShapeError`.
- branch-coverage, MEDIUM, fixed as proposed (merged with criteria-coverage LOW 4 and test-economy LOW 5): card 3 adds `auditFixRound` rows to `TestAuditClusterRound` for none, fewer, more, unreturned report and a violation, keeping one `TestEngine_Fix_Outcomes` violation row for the wiring.
- branch-coverage, MEDIUM, fixed as proposed: card 5 adds a twill row recording `fix_forks: 0`.
- branch-coverage, LOW, fixed as proposed: card 3 adds a single-fixer start refusal row keeping today's message.
- branch-coverage, LOW, fixed differently: proposed a row for a zero `FixKey`; instead card 5 restricts the recorded-name read to fix rows told `RequiresFans`, per D2's "their fix halves read none", so a twill fixer always carries a set or a `DefaultEngine` key and the branch cannot occur; card 7 states that a recipe `fix-fan` on a Webster, Merge or Darn row runs no twill fixer.
- branch-coverage, LOW, fixed differently: proposed a row for a recorded fan that no longer resolves; card 3 wraps `ResolveFan` errors in `validateFix` and `validateReview` with a way forward, lands a spec row and asserts it in `TestProfile_Validate`, and card 5 states the fix half carries it out.
- branch-coverage, LOW, fixed as proposed: card 5's `ProbeAndResume` gains a resumed-death row whose fresh spawn records the list's fan.
- branch-coverage, LOW, fixed as proposed: card 5's failed-write row asserts the round's review and acceptance record are archived.
- branch-coverage, LOW, fixed as proposed (merged with criteria-coverage LOW 3): card 8 adds empty-list rows for all four keys and a local-layer table refusing each of the four.
- branch-coverage, LOW, fixed as proposed: card 7 counts an all-empty wiring list as absent beside a recipe fan and adds a `[""]` row.
- branch-coverage, LOW, fixed as proposed: card 2 adds a row whose missing `burler-cluster-fan.md` returns the read error.
- criteria-coverage, LOW, fixed as proposed: card 5's per-round pick rows set both lists and assert the recorded `fix_fan` per round.
- criteria-coverage, LOW, disputed, every fix worse: the only target change is listing `internal/loomengine/config.go` beside its members on card 8, which `redundant-file-target` in `loom-plan-spec.md` refuses; card 8's Intent already names the file comment's rewrite.
- forced-edits, LOW, fixed differently: proposed listing `internal/burlerengine/prompt.go`'s file glyph, which `redundant-file-target` refuses beside `composeReviewPrompt`; cards 2 and 3 now name the file comment's update in their Intents.
- forced-edits, LOW, fixed as proposed: covered by the gate-quiescence fix, whose re-signed `gatedRowFanViolations` doc names the list type.
- forced-edits, LOW, fixed as proposed: card 6 counts four `writeRoundFacts` call sites.
- test-economy, LOW, fixed as proposed: cards 2 and 3 assert the filled lens bullets, the rubric path and a difference from the solo render, never a stencil's own text.
- row-reachability, LOW, fixed as proposed: card 6's rows merge a review half into the record so the `## Lenses` section renders a real table.
- row-reachability, LOW, fixed as proposed: card 3's fix-step subtest adds `self_check` to its optional list.
