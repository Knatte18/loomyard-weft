```yaml
lens: affected-set
outcome: reported
```

Task: affected-set lens. For each change a card makes, find what reads or asserts it, and which cards cover that set.

**Per change: the set found and the cards that cover it**

1. **Card 3 deletes webster's `StrandLive`, `TurnEndRead`, `TurnEndedAfter`, `turnEndSettle` and `awaitColdFixer`.**
   - Code callers: `recoverbatch.go` `awaitTerminal` (826/830), `reset.go:333` `refuseReportHeadBatch`, `planfixer.go:201` `FixPlanAwait`, `verifyfixer.go` `FixVerifyAwait`. Card 3 covers all of them.
   - Tests: `strand_test.go`, moved by cards 1 and 3.
   - Doc text: `websterengine/doc.go`, edited by card 3.
   - Also: `strand.go:1` file header and `recoverbatch.go:1` file header. No card covers either (finding F1).
   - `classify.go`'s `StrandLive` is a struct field, not the function, so it is unaffected.
2. **Card 3 deletes `RenderVerifyFixColdPrompt`, `WebsterPrefixVerifyFixCold` and the cold prefix stencil.**
   - Covered: `verifyfixer.go`, `stencils.go` `entries`, `roleOpeningStencils`, `markers_test.go:61`, the `template_test.go` helper and its three tests, `doc.go:560`, and the `webster-body-verify-fix.md` comment (all card 3).
   - Not covered: the `render.go` file and package comments at lines 1, 4, 25, 32 and 35, and the `template_test.go:375` comment on `testReplyPath` (finding F1).
   - The `docs/research/*` reports are dated research records, which are exempt.
3. **Card 10 re-signs `PlanTable`, `DiscussionTable`, `planLensDirective` and `discussionLensDirective`.**
   - Callers: `plan_test.go` `TestPlanTable`, `discussiontable_test.go` `TestDiscussionTable` and `TestDiscussionTable_Refuses`, `discussion_test.go` `TestProducerSpecs_SkillsAndParentDirective`, `loomcli/wiring.go` (wire), `plan.go`, `discussiontable.go`.
   - Card 10 covers all of them.
   - `loomcli/wiring_test.go` `TestWire_LanguageSkills` calls through `c.env.PlanTable()`, so it is unaffected.
4. **Card 13 re-signs `RenderForkPrompt` and `RenderRecoveryPrompt`.**
   - Callers: `beginbatch.go` (`BeginBatch`), `recoverbatch.go` (`recoverSpawn`), and in `template_test.go` the helpers `renderFork`, `renderForkWithLists`, `renderRecoveryWithUncommitted` plus `TestRenderPrompts_ParentDirective`.
   - Card 13 covers all of them.
5. **Card 13 adds a `{{.card_gate_fixer}}` marker to `webster-body-implementer.md`.**
   - Readers: `render.go:95` (fork) and `render.go:109` (recovery), both card 13, and `ImplementerBodyTemplate` at `render.go:69-73`, which no card names (finding F3).
6. **Card 12 re-signs `runnerCmd` and `runDetached`.**
   - Callers: `gatecli/cli.go:120` (`newCommand`) and `detachrun.go:262`. Card 12 covers both.
   - The `shedverbs/loopwait_test.go` hits are a different symbol.
7. **Card 7 adds the `Env.GateFixer` seam.**
   - Wirers and tests: `loomcli/wiring.go` and `wiring_seams_test.go` (cards 7 and 8), `battencli` `wire_seams_test.go` (card 7), `envkit.go` `nilLegal` (card 7).
   - `loomcli/landingdeps.go`, the `shedbuild` tests and the other `loomcli` tests build an `Env` and leave the seam nil, which is legal.
   - Covered.
8. **Card 7 adds the `fixer` key to a recipe gate element.**
   - Readers: the `shedrecipe` constructors (card 7), `loomrecipe` (card 10 `GateFixerSetting`), the recipe yamls (card 7).
   - Also: the `contracts/specs/shed-recipe-spec.md:72` element-key statement. No card covers it (finding F2).
   - `loomrecipe/sequence_test.go`, `loomcli/smoke_test.go` and `tools/tapestryrename` read the recipe files but pin gate names, not element keys.
9. **Card 4 adds `GateKey.Fixer` and `fixer_timeout_min`, and changes the template `card` line.**
   - Integration fixtures that write `card: {tiers: [0], scope: card}`: `websterengine/verifygate_integration_test.go:43`, `loomshed/verifygate_integration_test.go:143`, `gateslot/config_test.go:589-695`. They keep loading, because `gates` merges per key.
   - No test pins the template `card` line literally.
   - `docs/dotnet-hub.md:90` is an example that stays valid.
   - Covered by card 4 plus merge semantics.
10. **Card 9 adds `PreGoto` to loom's spec.** Loom had no earlier `PreGoto`; batten's `battenPreGoto` is separate. Covered.
11. **Card 12 adds the `fixing` phase.** The phase readers are `answerRunGoing` (`detachfront.go:866`, which tests only `phaseSlots`) and `runPhase.String`. Neither depends on a closed set. Covered.

**Findings**

- **F1 — MEDIUM, consistency.** Location: card 3, `webster-fixer-on-core`.
  - Card 3 deletes or moves the cold-fixer members but leaves file-level comments that name them.
  - Missed members:
    - `internal/websterengine/render.go` header and package comments at lines 1, 4, 25, 32 and 35. They name `webster-prefix-verify-fix-cold.md` and `RenderVerifyFixColdPrompt`.
    - `internal/websterengine/strand.go:1`. It says the file implements `StrandLive` and `TurnEndedAfter`.
    - `internal/websterengine/recoverbatch.go:1`. It names "webster's own classification machinery — …/TurnEndedAfter/StrandLive".
    - `internal/websterengine/template_test.go:375`. Its comment names `RenderVerifyFixColdPrompt`'s `replyPath`.
  - Card 3 lists only `websterengine/doc.go`, so the method covers part of the set.
  - Evidence: grep at `render.go:1,4,25,32,35`, `strand.go:1`, `recoverbatch.go:1`, `template_test.go:375`.
- **F2 — MEDIUM, consistency.** Location: card 7, `shedrecipe-fixer-element`.
  - The new optional `fixer` gate-element key falsifies the normative recipe spec, which describes each element as carrying only `attempts` and an optional `pass_on_cap`.
  - No card lists `contracts/specs/shed-recipe-spec.md`.
  - Evidence: `contracts/specs/shed-recipe-spec.md:72`.
- **F3 — LOW, consistency.** Location: card 13, `webster-fork-card-fixer-text`.
  - Card 13 adds `{{.card_gate_fixer}}` to `webster-body-implementer.md`, a stencil read by a third path, `ImplementerBodyTemplate`.
  - Card 13 names only `RenderForkPrompt` and `RenderRecoveryPrompt` and does not say how that path's consumer treats the new marker.
  - The affected set is not shown covered.
  - Evidence: `internal/websterengine/render.go:69-73` (`ImplementerBodyTemplate` reads `webster-body-implementer`), `render.go:95,109`.

**Tool calls**

1. Callers of the deleted and re-signed members (`refuseReportHeadBatch`, `TurnEndedAfter`/`StrandLive`, cold-prefix refs, `PlanTable`/`DiscussionTable`, `RenderFork`/`RenderRecovery`, `runnerCmd`/`runDetached`). The manifest had only the definitions, not every caller.
2. The enclosing test functions of the `PlanTable`/`DiscussionTable` calls, and other packages' `StrandLive` names, to map callers onto the tests card 10 lists.
3. `RenderForkPrompt`'s enclosing tests, readers of `webster-body-implementer`, `Env` wirers, phase readers, `PreGoto` users and the stencil-family list. These are one-hop readers the manifest lacked.
4. Docs, specs and fixtures naming the gate element keys, recipe files, the gate key field form and sibling config keys. This is non-Go readers, which the manifest did not cover.
5. The `shed-recipe-spec` line, fixtures pinning the `card` key literal, and `dotnet-hub` gate keys, to confirm the hits from call 4.
6. Every renderer of `loom-template-plan`/`loom-template-discussion` and `PartsPlanTable`'s signature, to check readers of card 10's new marker. The result did not show whether `PartsPlanTable` renders `planStencil`, so no finding was raised on it.
