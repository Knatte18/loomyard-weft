---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "The proposed `contract.takes` was left out of the places a note may go: `internal/loomshed/raddlefile.go:435` and `:453` copy it into the filed entry's record and body as the part's input. The stencil names `cost_line` and `next_best.line` only."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

## Changes

- F1: `UpsertTasksBatch` (`internal/boardengine/store.go`) checks the recipe against the entry the store held before the batch.
  `TestUpsertRefusesTheArrasRecipe` drives the batch write for a new and a darn entry.
- F2: the refusals in `internal/battencli/wire.go`, `internal/shedrun/seed.go` and `internal/shedcli/table.go` list the offered recipes only, through the new `shedrun.OfferedRecipeNames`.
  The arras recipe is still accepted.
  `internal/shedcli/table.go` is a listing the review did not name.
  The `profiles.yaml` header and the `split.reviewer_model` comment no longer name arras.
  `internal/configsync/testdata/profiles-flat.yaml` is unchanged: it is the former template the follow test starts from, not a copy compared with the current one.
  The tests that required arras in a refusal now require it absent, with `Loosened-Check` trailers on the commit.
- F3: the raddle stencil says where operator notes go when no operator is present (differently, see above).
- F4: `role-autonomy.md` names the raddle waiting line, and the Notices list in `orch-role.md` holds it.
- F5: the `land_notice` and `autonomy` texts agree with the new behaviour in the dobby template and every twin:
  `beamsetting.go`, `dobbyengine/config.go`, `dobbyengine/tasksetting.go`, the heddle glossary and the orch's autonomy-on line in `orchengine/prompt.go`.
- F6: `docs/overview.md` and the weaving glossary no longer offer arras as a board entry's recipe.
- F7: `internal/beamcli/dobbypairanchor_test.go` plants an unreadable child-recipe record and asserts the error naming the run.

Verification: `lym gate test` passed on tier0 and tier1 for the changed packages and their neighbours, and the F7 test ran in tier1.
The new test assertions were not run against the unfixed code.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency 1, MEDIUM (Notices section contradicts itself): fixed with the line as a bullet of the list.
- fix-consistency 2, LOW (autonomy twins stale): fixed.
- fix-affected-set 1, MEDIUM (`autonomyOnLine` stale): fixed.
- fix-affected-set 2, LOW (autonomy and land_notice twins stale): fixed.
- fix-proposal 1, LOW (autonomy twins stale): fixed, merged with the two above.
- fix-proposal 2, LOW (`contract.takes` leaks notes into the filed entry): fixed.
  In auto mode no reviewer reads the proposal, so a note there is only kept on record.
- fix-proposal 3, LOW (Notices section): fixed, merged with fix-consistency 1.
- fix-proposal 4, LOW (`profiles.yaml` header lists a closed set that the file's arras blocks break): fixed differently,
  by dropping the list ("one block per recipe it configures") rather than naming arras.

## Disputed

None.

## Deferred

None.

## Rulings

None.
