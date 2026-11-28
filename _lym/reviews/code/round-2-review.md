---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: "loomshed.SeedParentParam is exported with a doc comment and used at every production writer and reader of the seed's parent param: recordseed.go's reason, wiring.go's ReadSeed, sharedbootstrap.go's loomSeedFor and disagreeingSeedRefusal, arm.go's seed remedy and battencli/wire.go's childSeedParams. A grep of internal for a bare parent param literal finds no other production site in the diff."
  - id: F2
    status: resolved
    answer: ReadSeedErrorIsReturned now asserts both the step text and the way forward on the error message.
  - id: F3
    status: resolved
    answer: shed-recipe-spec.md now states the run-ending block rule, the widened clause 1 with LoomSeedEngine and the seed-after-record order, and names the recipe files under contracts/recipes.
  - id: F4
    status: resolved
    answer: "TestRecordSeedDeps_ReadsTheRunsSeed writes a real seed and asserts ReadSeed's recipe, driver and parent and the slug against literals, and checks that SeedPath names the seed file the run wrote."
  - id: F5
    status: resolved
    answer: "The door is one-way, and the prose names both records a reverted binary cannot use. The one-word radius `runs` is what the final-summary spec allows, and the prose names loom, shed and batten."
  - id: F6
    status: resolved
  - id: F7
    status: resolved
    answer: loomshed.go, doc.go and smoke_test.go no longer say tapestry-recipe.yaml holds the rows.
  - id: F8
    status: resolved
  - id: F9
    status: resolved
    answer: "keptRecordSupportLog's line is true on the only path that reaches it: the record exists, since a missing record with no stored record returns Stuck before the log branch. A test pins its bytes."
  - id: F10
    status: resolved
    answer: TestExpand gains the admitted nested run-ending row and the refused not-last inner block row.
  - id: F11
    status: resolved
    answer: "CancelDuringTheCallWinsOverStuck cancels inside ReadStored. In recordwrite.go no cancel check sits between ReadStored and the noStoredReason branch, so the test reaches the new cancelErr before Stuck. Removing that check would fail the test."
  - id: F12
    status: resolved
---

Round 2 is incremental.
I read every diff piece, recordwrite.go whole and the changed hunks the changes file lists.
I ran `lym gate test ./internal/loomshed ./internal/shedbuild ./internal/loomcli`: 2524 passed, 0 failed.
Every prior finding is resolved and no new BLOCKING defect sits on a changed hunk, so the verdict is APPROVED.

## Focus answers

- **Seed parent param.**
  It is one constant, `loomshed.SeedParentParam`, at every production writer and reader the diff touches.
  The test in `wiring_test.go` spells `"parent"` as its independent expected value, which is right for a test.
  `internal/shedcli/seed.go`'s help text still spells `parent=main`, but that file is outside the review range and is static usage prose.
- **Record-Seed's refusals.**
  Both Stuck reasons are reached and their text asserted: the Discussion goto, and the batten and re-seed sequences with slug, seed path, driver and parent filled in.
  The seed-read error asserts its way forward, and the cancel path before the stored-only Stuck is reached and covered.
- **`recordSeedDeps`.**
  The test pins recipe `tapestry`, driver `llm`, parent `feature-x` and slug `pair` against literals, and checks that `SeedPath` is the seed file `WriteSeed` created.
- **The missing-support-log branch.**
  In the stored-only mode that branch runs only over an existing record.
  A missing record with no stored record returns Stuck before it, and a found stored record takes the stored-record log.
  So "the record was kept as it stood" states only what happened.
- **Specs and headers.**
  The spec's block rule matches `checkRunEndingBlock`: last of its own recipe and gated `always`, applied to the resolved exit so nested blocks are held too.
  The spec's clause 1 matches `recordSourceRow`.
  The headers of the tapestry, jacquard, motif and arras recipes and the loomshed comments name the files that now hold each row.
- **Change description.**
  The door is one-way.
  The prose names the two records a reverted binary cannot use and the three modules at stake.
  The board does not validate a recipe name against the vocabulary, so a board entry naming `jacquard` stays readable by an older binary, and `runs` covers the radius.
  The description names no tooling outside the repository's own subjects.

## Rejected

Each of these is below the incremental round's BLOCKING bar, so none is raised:

- `contracts/recipes/jacquard-recipe.yaml`, in Code-Burler-Review's comment: "fix-scope: source, the only row in the recipe still carrying this value now that Discussion-Burler-Fix and Plan-Burler-Fix both run overlay".
  The comment names a row jacquard does not hold, and Code-Burler-Fix runs source too.
  It is LOW wording, moved verbatim.
- `internal/loomcli/wiring.go`: no test pins that `wire` passes `loomrecipe.EngineRow(c.recipe, discussionWriteEngine)` as `DiscussionRow`, which picks between the two Stuck ways forward.
  `TestRecipeKeys` pins `EngineRow` itself for tapestry and jacquard, so a regression there needs a wrong argument at the call.
  It is LOW.
