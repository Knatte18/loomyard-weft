---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: internal/loomrecipe/names.go:14
    summary: the RecipeEngines doc comment folds a relative clause into its list of steps, so the clause and the following step read as one
---

Range reviewed: `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main`, no `_lyx/loom/rework` round exists), covering cards 1-9 and both round-1 fix commits.
I read every changed production file and the tests behind each card, and ran the package tests of every touched package plus `cmd/lyx`, `internal/loomcli`, `internal/webstercli` and `internal/shedverbs`; all pass.

### [NIT] RecipeEngines doc comment reads as one tangled sentence

**Location:** `internal/loomrecipe/names.go:14`
**Issue:** The comment lists the function's steps as "collects each row's engine name, adds `discussionSeatsEngine`, which New substitutes for the Discussion-Write row's engine and so a loom row can reach, de-duplicates, and returns the result sorted."
The relative clause after `discussionSeatsEngine` is ungrammatical ("and so a loom row can reach") and runs straight into the next list item, so "de-duplicates" reads as part of the clause.
**Fix:** End the list before the clause, for example: "collects each row's engine name, adds `discussionSeatsEngine`, de-duplicates, and returns the result sorted. `New` substitutes `discussionSeatsEngine` for the Discussion-Write row's engine, so a loom row can reach it."

## Focus verdicts

Neither focus entry is a defect.

1. `LoadConfig` in `internal/loomengine/config.go`, the `modelspec.LoadRegistry` failure for a non-empty `discussion_advisors`: not a defect.
   `TestLoadConfig_Refuses` ends with the subtest "a malformed models.yaml refuses advisors and is never read for none".
   It plants a malformed `models.yaml` and a one-entry `discussion_advisors` list, then asserts `LoadConfig` fails and the error contains `discussion_advisors` and the way forward "fix models.yaml, or set the key to an empty list for no advisors".
   It then rewrites the key to `[]` and asserts `LoadConfig` succeeds against the same malformed registry, which shows an empty advisor list never reads the registry.
   The refusal has its row in `contracts/specs/refusal-spec.md` ("config: models.yaml unreadable for discussion_advisors"), so `PATTERN-refusal-way-forward` holds.
2. `Env.ParentReview` and `Env.CommitDiscussion` in `internal/shedrecipe/recipe.go`, and the `internal/shedrecipe/entries_gate.go` header: not a defect.
   They now name the readers as "the entries that wrap a discussion producer" and "every entry that wraps a discussion producer", and the header names "every gate-capable entry", so none enumerates entries the `DiscussionSeats` entry also invalidates.
   `Env.DiscussionSpec` still names the `DiscussionWrite` entry alone, which is correct because `DiscussionSeats` never reads it.

## Notes on what I checked and did not flag

- `composePrompt` in `internal/seatengine/prompt.go` is now called only from tests.
  Card 1 keeps it on purpose ("`composePrompt` keeps its signature and callers", additive changes only), so it is not a finding.
- The `ParentReviewBrief`, `NewDiscussionGate` and `TestLoadConfig_Refuses` "names no entry index" additions, and the `unboundedAskOperator` check in `internal/loomengine/discussiontable_test.go`, are each required by their card's text, so I raised no test-economy finding against them.
- No card carries a `Rename` or `Delete` group, so the per-type mechanical checks of the plan spec have nothing to run.
- Each new production file and changed doc comment follows the repository's comment conventions; I found no end-of-line Go comments and no empty intensifiers.
- The new roles, the `DiscussionSeats` entry, the config keys and the stencils are registered in `roleOpeningStencils`, `shedrecipe.registry`, `ConfigOpenMaps`, the coverage guard and the test budget, and the docs landed in the same commits.
