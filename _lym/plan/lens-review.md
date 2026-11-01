# Lens review

Lenses: coverage, row-validity, forced-edits, guard-reach, way-forward; every fork returned a report, saved beside this file as `lens-review.lens-<lens>.md`.
Duplicates across lenses are merged onto one line, naming each lens and its finding number.

- row-validity 1, BLOCKING — `TestValidatePart` and loomshed `TestJoinParts` over a fake index cannot produce `handle-dangling` or `part-spot-target`: fixed as proposed; card 7's tests run over `planglyph.NewIndex(fabricengine.NewReferenceRule())`, as `TestNewPlanGate` does (`internal/loomshed/gates_test.go:397`).
- row-validity 2, MEDIUM — the `caller-uncovered` half of `TestValidateFormat_PartScope` would go through the dispatch form, which runs no `planGatePass`: fixed as proposed; card 5 makes it a row of `internal/planglyph#TestGlyphChain_CallerCoverage`, listed under Edit, and the status half calls `ValidateFormat`.
- row-validity 3, coverage 2, guard-reach 1, MEDIUM — the `--part` "no PartsPlanWrite row" refusal is unreachable behind `refuseVerbOffRecipe` (`internal/loomcli/arm.go:186-198`): fixed as proposed; card 18 drops the branch and its darn row, and adds an absent-parts-file row instead.
- row-validity 4, MEDIUM — `TestEngine_Run_WriterEndDecidesTheJoin` has no deterministic order: fixed differently; the proposed release-then-wait sequence still leaves the join's select racing, since the ender's signal is ready before the chair's wait goroutine reports, so card 10's test calls `Engine.join` directly with the ender already fired and the chair already released, and the rule marks the writer in both select cases, so the outcome holds whichever case wins.
- row-validity 5, LOW — writer end rows repeat `TestEngine_Run_RequiredAdvisorEndsTheStep`: fixed as proposed; cards 9 and 10 add writer variants of that table's rows, and the new tests keep only the done line, no up line, and the resume line with outputs present.
- row-validity 6, LOW — the "writer asked a fix" resume row adds nothing: fixed as proposed; dropped from card 10.
- row-validity 7, LOW — "an Accept that is the told one" is not comparable: fixed as proposed; card 14's told `accept` hook records its part and returns an Accept recording its call.
- row-validity 8, forced-edits 9, LOW — `TestDecisionIDs` named a file that does not exist: fixed differently; card 2 puts it in `validate_test.go` beside `internal/discussionparser#TestCheckDecisionIDs` rather than declaring a new file, since that file holds the package's decision-id tests.
- forced-edits 1, BLOCKING — `internal/shedrecipe#TestNames` pins the registry names: fixed as proposed; card 16 lists and extends it.
- forced-edits 2, BLOCKING — `internal/shedrecipe#minimalEntries` needs a row per registry entry: fixed as proposed; card 16 lists and extends it.
- forced-edits 3, BLOCKING — `internal/loomrecipe#gatedRowFanViolations` refuses a gated row of another engine: fixed as proposed; card 17 admits both engines and updates the message.
- forced-edits 4, MEDIUM — the timing pin is `internal/reportengine#loomRowPhases`, not `TestComputeTiming`: fixed as proposed; card 17 targets `loomRowPhases`.
- forced-edits 5, LOW — `internal/shedrecipe#gateCapableEntries` misses the new gated entries: fixed as proposed in card 16.
- forced-edits 6, LOW — `Table.Timeout` and `Result.Advisors` doc comments go stale: fixed as proposed; card 9 lists `internal/seatengine#Table` and `internal/seatengine#Result` and rewords both.
- forced-edits 7, LOW — the `MultiLLMProducer` doc says advisors are never judged: fixed as proposed in card 11.
- forced-edits 8, LOW — `internal/loomrecipe/doc.go:15` places Drift-Baseline before Plan-Write: fixed as proposed in card 17.
- way-forward 1, guard-reach 6, MEDIUM — the part check's `CanonicalizeHandles` may respell an export, so the join's exact match refuses with a way forward that cannot clear it: fixed differently; no card proposed a change, so card 7's `ValidatePart` re-parses after the check and refuses `part-export-missing` at the writer's own gate, card 14's writer stencil sends that finding to the chair, and the chair stencil respells the export in the parts file and tells every using writer, so the join's exact match then holds.
- way-forward 2, LOW — card 11 overstated the retry of `ErrAgentDied`: fixed as proposed; card 11 states `internal/shedtransient`'s actual classification and names `lym loom resume` in the died error.
- way-forward 3, LOW — the missing-parts-file refusal reaches a writer who cannot run goto: fixed as proposed; card 7's error routes it through the parent, and card 14's writer and chair stencils say to report it.
- way-forward 4, LOW — `record-missing` spends the gate's re-prompts: fixed as proposed; card 3 returns an absent record as an error naming `lym loom validate-discussion`.
- way-forward 5, LOW — no message for an absent status in `join-parts`: fixed as proposed; card 19 gives it the "chair's alone" message.
- way-forward 6, LOW — a failed table build on resume leaves live strands: fixed as proposed; card 14's refusals name `lym reed remove --name <strand>` for the parent first.
- way-forward 7, coverage 14, LOW — a half-finished `NewPartsPlanRotator` loses the prior-plan block on retry: fixed differently; card 15 archives the parts first and rotates the plan last, so every failure leaves the plan in place and the retry announces it, with a row for a failing `archiveWebster`, rather than adding a row for the old order.
- way-forward 8, LOW — the `parts_max` refusal names no way forward: fixed as proposed in card 1.
- way-forward 9, LOW — a mid-join write failure has no way forward and the chair stencil seemed to forbid a rerun: fixed as proposed; card 6's error names a rerun and card 14's chair stencil allows it after a failed join.
- coverage 1, guard-reach 2, BLOCKING — the chair's default and the set-value check ignore a 0 `plan_timeout_min`: fixed differently; the proposal to compare at config load cannot read shuttle's `run_timeout_min`, so card 1 adds `Config.PlanTimeout(runTimeout)` and `Config.PartsChairTimeout(runTimeout)` returning an error, card 14's table takes `runTimeout` and fails on that error, and card 16 tells it shuttle's value; a Shared Decision records it, and rows cover the 0 case.
- coverage 3, MEDIUM — the real commit pathspecs are untested: fixed differently; card 16 adds `partsPlanCommitPathspec` and rows in `internal/loomcli#TestPlanCommitPathspec` for its pathspec, webster's directory included, rather than a git round trip, since the commit-on-Done-and-Stuck rule is already pinned by `internal/loomshed#TestPlanWrite_Call` and the committing call by `CommitAnchoredPaths`.
- coverage 4, MEDIUM — `NewPartPlanGate` and `NewPartAccept` have no row: fixed as proposed; card 7's `TestValidatePart` gains rows for both, Accept over a zero result.
- coverage 5, guard-reach 3, MEDIUM — `ValidatePart`'s absent, malformed and unparseable-part branches have no row: fixed as proposed in card 7.
- coverage 6, MEDIUM — `JoinParts` refusing a missing parts file has no row: fixed as proposed in card 7.
- coverage 7, MEDIUM — no row for re-signed or deleted exports, nor for the carried overview sections: fixed as proposed in card 6.
- coverage 8, MEDIUM — "the block names no step" is unchecked: fixed as proposed; card 12 asserts the block's default holds no `Step `.
- coverage 9, MEDIUM — the prompt-content check is weak and the glyph subsection moves: fixed differently; card 12 drops its new `### Plan-wide rules` heading and pins the composed prompt's line multiset against a golden captured before the move; the subsection's move is kept, since one block cannot sit at two positions (`internal/seatengine#validateStencil` includes a block whole, one level deep), recorded as a Shared Decision.
- coverage 10, MEDIUM — no flag-on run row: fixed as proposed; card 17 adds `TestSequence_PlanSplitRunsThePartsRows` over `buildSequenceFixture`.
- coverage 11, LOW — no split-recipe goto row: fixed as proposed; the same test moves the run onto `Plan-Write` with goto and steps it.
- coverage 12, guard-reach 4, MEDIUM — `segmentWriter`'s walk is tested one step deep and can run past the writers: fixed differently; card 20 steps at most once, which is all D1 asks, with a row for both writers latest-Skipped, instead of rows for a multi-step walk.
- coverage 13, LOW — no `handle-collision` row for a handed-in handle: fixed as proposed in card 4.
- coverage 15, LOW — no rows for the `parts` gate's refusals: fixed as proposed; card 16 adds them to `internal/shedrecipe#TestResolveGateSpec_Rejections`.
- coverage 16, LOW — `writerExit` has no default branch: fixed as proposed; card 11 maps any other end to an error naming the seat and outcome, covered by its one-row-per-case test.
- coverage 17, LOW — four branches without rows: fixed as proposed for the unreadable fragment, the friction render failure (card 13) and the `validate-parts` config error (card 18).
- guard-reach 5, LOW — "unknown depth" and "gates error" rows test branches an upstream check rules out: fixed as proposed; cards 13 and 14 state them as fail-loud wiring guards and drop their rows.
