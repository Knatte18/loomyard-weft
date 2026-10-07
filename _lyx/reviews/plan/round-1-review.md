---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/06-two-agent-round.md (Edit list) and internal/burlerengine/template_test.go:87
    summary: Card 6 deletes the exported BurlerTemplateRoundOrchestrator embed but internal/burlerengine/template_test.go still references it and is not a card 6 target, so the burlerengine test package stops compiling after card 6
  - id: F2
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/03-per-round-models.md (Intent, LoadConfig paragraph)
    summary: Card 3 adds loom config load refusals (empty list, unparseable entry, mapping value) with no refusal-spec row and no stated way forward, which PATTERN-refusal-way-forward requires in the same commit
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/06-two-agent-round.md (Create list, Seams paragraph, Tests paragraph)
    summary: StrandRemover and its reed-backed implementation are a public cross-package symbol hidden inside the engine-split card, and the card plans no test for the real implementation
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/plan/06-two-agent-round.md (Intent)
    summary: Card 6 introduces the ready-marker handoff, the post-handoff review comparison and moves caller gates to the fixer only, but its Intent never states what these can let through or what bounds them
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/01-shuttle-probe.md (Tests paragraph)
    summary: Card 1 plans ProbeGated tests for the not-found removal and the refusal answers, which TestAttach_RemovesSupersededStrands and TestAttach_Multiplicity already cover through the shared attach body
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/05-round-stencils.md (step file header comments paragraph)
    summary: Card 5 retargets the header comments of steps 1 and 2 but not step 3, whose header keeps naming the round orchestrator that card 6 deletes
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/plan/03-per-round-models.md and _lyx/plan/02-ready-marker-path.md (target lists)
    summary: docs/overview.md line 438 enumerates loom.yaml role keys and the module tree lists every internal package, yet no card updates it for the fix key, the list form or the new internal/burlermarker package
  - id: F8
    severity: LOW
    class: scope
    location: _lyx/plan/04-await-review-verb.md (Behaviour and Tests paragraphs)
    summary: Card 4 specifies exit 1 envelopes for a missing argument and an unreadable stat but plans no test row that reaches either
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/plan/07-two-half-resume.md (Tests paragraph, burler_test.go row)
    summary: The test row says each one-live and done/gone combination removes the live strand, but a done/gone pair has no live half and nothing to remove
---

### [BLOCKING] Card 6 breaks `internal/burlerengine/template_test.go`

**Location:** `_lyx/plan/06-two-agent-round.md`, Edit list; `internal/burlerengine/template_test.go:87`.

**Issue:** Card 6 deletes `contracts/stencils/burler/burler-template-round-orchestrator.md` together with its embed, registry entry and claims, which removes `stencils.BurlerTemplateRoundOrchestrator`.
`internal/burlerengine/template_test.go` (package `burlerengine`) reads that variable in `TestTemplate_FillsWithAllMarkers`'s `orchestrator` row and carries `orchestratorMarkerValues`, with the old orchestrator's four markers including `instruction_3_path`.
The file is not a target of any card, so after card 6 the package's tests do not compile, and the card fails its own "builds on its own" rule.
The same file is also the existing test that proves each shipped round prompt fills with its required marker subset and fails naming a missing one.
The two new orchestrators bring new required markers (`review_format_path`, `ready_marker_path`, `instruction_3_path` on the fixer only), and the card's plan covers them only through `prompt_test.go`'s "markers render" assertion, which does not do the missing-marker sweep.

**Fix:** add `internal/burlerengine/template_test.go` to card 6's Edit list.
Replace the `orchestrator` row and `orchestratorMarkerValues` with a row and a values map per new orchestrator, so the fill and missing-required-marker sweep covers both.
Add one clause to the Tests paragraph saying so.

### [BLOCKING] Card 3's new config refusals carry no spec row or way forward

**Location:** `_lyx/plan/03-per-round-models.md`, Intent paragraph on `LoadConfig`.

**Issue:** `LoadConfig` now refuses an empty `review`/`fix` list, an unparseable entry and a mapping value.
`lyx loom` verbs load this config, so these are refusals reachable through a bound module's verbs.
`PATTERN-refusal-way-forward` requires every such refusal to name its way forward in the message and to land its row in `contracts/specs/refusal-spec.md` in the same commit, with a test that reaches it.
The loom section already holds the sibling row `config: review key below 1`, so config-validation refusals belong in that table.
Card 3 names the key and entry index in the message, but states no way forward, and edits no spec file.

**Fix:** state in the Intent that each of the three messages names the key and entry and tells the operator what to set (a model-spec, or a non-empty list of them).
Add `contracts/specs/refusal-spec.md` to the Edit list and one row per refusal (or one row covering the list errors) in the loom section.
The `config_test.go` rows the card already plans are the tests that reach them.

### [MEDIUM] StrandRemover is hidden inside the engine-split card

**Location:** `_lyx/plan/06-two-agent-round.md`, Create list, Seams paragraph, Tests paragraph.

**Issue:** The granularity rule gives a symbol that is part of a module's public surface, or reused across packages, its own card.
`StrandRemover` and its reed-backed implementation (`remover.go`) are that: the engine takes one, `burlercli` and `loomcli` build one, and `shedadapters` is told the same one in card 7.
It is also a self-contained unit: a no-op for a strand reed does not report live, an error for a failed liveness probe or a failed removal, and a Warn log for every removal.
Card 6 nonetheless folds it into a change that rewrites `Engine.Run`, `Result`, `RunOpts`, `Profile`, prompts, stencils and five caller packages.
The card's Tests paragraph also plans only a fake remover, so the real implementation's three behaviours have no test.
`internal/websterengine/strand_test.go`'s `TestRemoveStrandIfLive` shows the shape those tests take against a reed fake.

The other symbols card 6 creates (`Handle`, `RunnerShuttle`, `ErrHalfNotStopped`, `join`) do not need their own cards: the `Shuttle` seam change, the `Result`/`RunOpts`/`New` signature changes and every caller must move in one commit, and `join` and `ErrHalfNotStopped` have no use apart from the new `Engine.Run`.

**Fix:** insert a new card before the engine split (renumbering the engine card and the resume card, and every reference to them in the plan) that creates `plan:internal/burlerengine#StrandRemover`, its reed-backed implementation and constructor, `remover.go`, and a `remover_test.go` that covers not-live, failed probe, failed removal and live-removed over a reed fake.
The engine card then lists `StrandRemover` under `Uses` instead of `Create`, and its Tests paragraph drops the remover implementation.

### [MEDIUM] Card 6 states no bound for the handoff and the moved gates

**Location:** `_lyx/plan/06-two-agent-round.md`, Intent.

**Issue:** The rubric requires a card that introduces a routing edge, or removes, downgrades or makes bypassable a guard, to state in its `**Intent:**` what the change can now skip or let through and what bounds it.
Card 6 introduces the Go-written ready marker as the one edge that releases the fixer, the post-handoff comparison that covers the review file only, the "fixer done before the marker" error and the `ErrHalfNotStopped` no-retry exit.
It also downgrades guards: the single-session A-before-B ordering becomes a prompt-enforced rule on a separate session, and the caller's gate entries no longer re-prompt the session that wrote the review.
Cards 1, 4 and 7 each carry such a paragraph; card 6, the card that makes these changes, carries none.
The decision record states the bound (D5, D7 and Open risks): the marker is written only after the reviewer's done outcome, fork audit and strict parse for the current attempt, the comparison covers the review file only, and the fixer's write surface and prompt are what keep it from creating the marker or writing another round file, which review alone catches.

**Fix:** add a paragraph to card 6's Intent, in the shape of cards 1, 4 and 7, stating what the handoff and the comparison cannot catch (a fixer that creates the marker, edits before it, or writes a round file other than the review) and what bounds each (the write surface in the fixer stencil, the review-file comparison, the lyx-only marker write after the current attempt's checks, `Run` removing a stale marker first).

### [MEDIUM] Card 1 re-tests behaviour the shared attach body already covers

**Location:** `_lyx/plan/01-shuttle-probe.md`, Tests paragraph.

**Issue:** `ProbeGated` is the body of `attach` stopping before `Wait`.
Of the three planned rows, two (the not-found answer removes a live superseded strand exactly as `AttachGated` does, and an errored or doubly-attachable candidate set returns `AttachGated`'s error) exercise code that the card does not touch.
`TestAttach_RemovesSupersededStrands` covers the removal, and `TestAttach_Multiplicity` (with its `ErrorCandidateDominatesAttachableCandidate` and `TwoAttachableCandidates_Errors` rows) covers the refusals.
Only the found-candidate-returns-an-unwaited-handle row is new behaviour.

**Fix:** reduce the planned test to the found-candidate behaviour: a non-nil handle, `found` true, `StrandGUID()` and `RunDir()` naming the candidate, no wait having run, and `Wait` then classifying done once the output file exists.
Keep the not-found `(nil, false, nil)` shape as an assertion in that same test only if no existing test pins the nil handle, and say that the removal and refusal paths are covered by the existing attach tests.

### [LOW] Step 3's header comment keeps naming the deleted orchestrator

**Location:** `_lyx/plan/05-round-stencils.md`, paragraph on steps 1 and 2.

**Issue:** `burler-step-1-explore.md`, `burler-step-2-review.md` and `burler-step-3-fix.md` each name `burler-template-round-orchestrator.md` in their header comment.
Card 5 retargets the first two and edits the third's body, but never its header, and card 6 deletes that orchestrator.

**Fix:** extend the card 5 sentence to step 3: its header names the fixer orchestrator as the one that directs it.

### [LOW] docs/overview.md is not updated for the fix key or the new package

**Location:** `_lyx/plan/03-per-round-models.md` and `_lyx/plan/02-ready-marker-path.md`, target lists.

**Issue:** `docs/overview.md` line 438 lists loom's config keys ("the `discussion`/`plan`/`review`/`judge` role model-specs") and describes `review` as one model-spec; after card 3 there is a `fix` key and `review` may be a per-round list.
The same file's module tree lists every `internal/` package, and card 2 adds `internal/burlermarker`.
CLAUDE.md requires the docs to land with the change.
Cards 4 and 6 edit `docs/overview.md` for the burler entry only.

**Fix:** add `docs/overview.md` to card 3's Edit list with the config-key sentence, and add the `internal/burlermarker` tree line to card 2 (which then needs `docs/overview.md` in its target list as an Edit).

### [LOW] Card 4 specifies error exits it never tests

**Location:** `_lyx/plan/04-await-review-verb.md`, Behaviour and Tests paragraphs.

**Issue:** The Behaviour paragraph defines `output.Err` with exit 1 for a missing argument and for a stat error other than not-exist.
The Tests paragraph lists four rows (present, absent after cap, created while polling, nothing written), none of which reaches either error.

**Fix:** add a missing-argument row to the verb's test.
Add a stat-error row (for example, a marker path whose parent is a regular file) if it is portable, or drop the unreadable-stat clause from the Behaviour paragraph.

### [NIT] Card 7's done/gone test row cannot assert a removal

**Location:** `_lyx/plan/07-two-half-resume.md`, Tests paragraph.

**Issue:** The row "each one-live and done/gone combination removes the live strand and spawns attempt 1" includes the pair where one half is done and the other gone, which has no live half, so no strand is removed.

**Fix:** word the row as: a pair with one live half removes that strand and spawns attempt 1; a done/gone pair removes nothing and spawns attempt 1.

## Focus verdicts

These settle the directive's questions that did not become findings.

- **Card 6 granularity:** one card is justified for the engine split and its callers, since the `Result`, `RunOpts`, `Profile` and `New` changes and the `Shuttle` seam change leave no building tree in between; `ModelChoice` and `RoundModels` are already their own card (3).
  The one hidden public-surface symbol is `StrandRemover`, raised as F3.
- **Decision coverage:** D1 to D12 and every Constraint trace to a card (D8 to cards 5 and 6, D9 to cards 3 and 6, D10 to card 3, the docs list to cards 3, 5, 6 and 7), with the gaps raised as F2 and F7.
- **Scope:** no card edits a file or adds behaviour the decision record does not license; the worktree `_lyx/config/loom.yaml` is untouched, and the `configreg` open-map edit is the "reconcile behavior" the Notes license.
- **Test economy:** only card 1 duplicates existing tests (F5).
  The cards 3, 4, 6 and 7 tests extend existing tables or pin new behaviour; card 7's resumed failure rows exercise the resume entry's wiring into the shared `join`, and are acceptable as rows beside card 6's table.
- **Attack surface:** cards 1, 4, 5 and 7 state a bound; card 6 does not (F4).
- **Verify coverage:** every targeted package is under `./...`, the `integration` tier is run, `tmux` and `llm` are compiled, and the section follows the template's order.
- **Card 6 alone, and card 7:** after F1 is fixed the tree builds and tests pass after card 6.
  The retained pair-file attach probe then matches no run (neither half declares both output files) and is inert until card 7 replaces it, which is acceptable inside one plan.
  Card 7's `Uses` handles all resolve to cards 1, 3 and 6, and D6's resume rules are fully carried by card 7.
