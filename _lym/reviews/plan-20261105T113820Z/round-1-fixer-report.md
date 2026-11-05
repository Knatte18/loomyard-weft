# Fixer report — plan, round 1

The decision-class findings F3 and F8 went to the parent and are left out of the list below.
The parent recorded both as Added after Discussion entries, and the LOOM mail said to act on them, so I applied them:
- the project's `type` key in cards 10, 12, 16 and 22;
- the citation of the decision in the `filing-level` Shared Decision and in card 16.

## Changes

- **F1** (fixed as proposed): card 13 now declares `shedrun.RecipeRaddle` in place of `boardengine.SketchRecipe`, and its Intent says `boardengine` compares against it with no import cycle. Card 23 drops its declaration, lists the constant under Uses, and says `RecipeNames` takes the constant card 13 declared.
- **F2** (fixed differently): the review offered two options. I kept the label rule first, as the review's first option allows, and made its refusal name `lym board promote <slug> --recipe raddle` when the entry's prior state is a discussed note. To make the prior state visible, `validateLabels` is re-signed to `validateLabels(t Task, prior *Task, v Vocabulary)` and `Store.validateWrite` passes the snapshot's entry. The `TestDiscussedPromotion` rows for a plain promote of a labelled Sketch and for an upsert-to-darn of a Sketch now assert the `--recipe raddle` way forward.
- **F4** (fixed as proposed): card 12 gains the `decision-unknown` and `pattern-unknown` checks, each with a `TestCheck` row. Card 22's step 2 makes a `SplicePart` error `Stuck` with `Halt`, naming the error and the way forward `lym loom goto Raddle-Write`, and gains a `TestRaddleFile` row for it.
- **F5** (fixed as proposed, first option): card 14's Bound now says only a raddle entry is caught, by card 24's guard. It also says `lym board raddle <slug>` and `lym report raddle` then name the old proposal.
- **F6** (fixed as proposed): card 21's stencil renders `{{.parent_directive}}` and `{{.edit_directive}}` as the discussion stencil does. `contracts/stencils/parentdirective_test.go` joins its Uses.
- **F7** (fixed as proposed): `internal/loomcli#arbiterRecipeDescription` joins card 23's Edit list. Its Intent describes a recipe with a `RaddleFile` row, and `TestRecipeKeyedWiring`'s raddle row asserts the description.
- **F9** (fixed as proposed): card 22's step 3 renders each part's body (the Sketch slug, the contract, the packages with their sizes), and `TestRaddleFile`'s tasks row asserts it.
- **F10** (fixed as proposed): `internal/loomcli/doc.go` is dropped from the Uses of cards 5, 6, 18 and 20 and moved from card 23's Uses to its Extends. Card 23's Intent gains the sentence on the RaddleFile ending.
- **F11** (fixed as proposed): `internal/boardengine#StatusDraft` joins card 13's Edit list, with the Intent sentence naming the released exception.
- **F12** (fixed as proposed): card 27 names `raddle` and `raddle_auto_approve` in the overview's loom key list, and the `report` module and `cost_min_runs` in its report module line.
- **F13** (fixed as proposed): the label rule judges only a write that adds the label, makes the entry a task or project, or changes its recipe, so an untouched legacy task keeps taking writes. `TestValidateLabels` gains the legacy rows.
- **F14** (fixed as proposed): card 13 gains a Bound. The replayer skips `checkDiscussedPromotion`, and card 24 does not catch the result.
- **F15** (fixed as proposed): card 24's pass rationale now names a fresh `lym batten run` after a cancel that removed the status, past a filing.
- **F16** (fixed as proposed, with the mechanism mine): `Deps.Notify` becomes `func(line string) (posted bool, err error)`, and `pawlDeps` returns `NotifyOrchAt`'s posted result. A hand-off post with posted false halts with no marker written, and a `TestBeamUp` row covers it. The re-signed `Notify`'s other sites join card 25's Edit list: `awaitRun.notify`, which ignores posted, `validPawlDeps` and `envkit.FullEnv`.
- **F17** (fixed as proposed): `slug-taken` exempts the run's own entry, which `slug-sketch` judges.
- **F18** (fixed differently): the proposal said one rule each still moves into `PATTERN-shed-recipe-registry.md` and `PATTERN-test-speed.md`. Both are already stated there: the off that reads more than one key at `pattern/PATTERN-shed-recipe-registry.md:29`, and the OS-skip rule at `pattern/PATTERN-test-speed.md:119`. So no rule moves at all, and the three pattern files leave card 4's Extends. Card 4 records the line citations and the teardown lock's takers in place of the conditionals, and the overview's `pattern-headroom` decision now says the same.
- **F19** (fixed as proposed): the nil-hook row is dropped from `TestRunner_SessionEnd`.
- **F20** (fixed as proposed): `internal/loomrecipe#AmendmentTarget` joins card 23's Edit list, and its doc comment is named beside the sibling comments.

## Self-check

- fix-consistency, LOW: the overview's `pattern-headroom` decision still said rule text moves. Fixed as proposed.
- fix-affected-set, LOW: `draft.go`'s file header and the glossary's `drafter` and `draft` rows still said every new task is draft. Fixed as proposed, in card 13 and card 27.
- fix-claims, LOW: card 4's `PATTERN-test-speed.md` line spans were off. Fixed as proposed.
- fix-claims, LOW: "through `validateWrite` alone" overstates, since the replayer also runs `checkRunLocks`. Fixed as proposed in cards 13 and 14.
- fix-proposal, LOW: the label rule's raddle way forward misleads for a discussed note with a missing record. Fixed as proposed, through the fork's second option: a Bound naming the two-step lead through card 24's no-record refusal. The label rule cannot see the record file.

## Disputed

None.

## Deferred

None.
