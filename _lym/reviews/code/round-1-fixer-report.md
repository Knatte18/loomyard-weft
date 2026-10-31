# Fixer-report, code review round 1

## Changes

Every finding is fixed as proposed unless marked.
Each fix is its own commit, run green through `lym gate test` for the packages it touched.

- **F1** `92d232f9`: `TapestryRetiredRows` holds `Loom-Preflight` → `loomshed.NameTapestryPreflight`.
  The doc comments of the function, `internal/loomrecipe/doc.go` and `contracts/recipes/tapestry-recipe.yaml` name the entry.
  `TestRenameRetiredRows` has a row for it.
- **F2** `c428e1b2`: fixed with an own change to the second location.
  `TestValidateRecipe` has a `loom` row and asserts the error text holds `unknown recipe` and `tapestry`.
  The review named `internal/shedcli/cli_test.go`, but the test that calls the seed-writing lookup with a bogus `--recipe` is `TestWriteSeed_UnknownRecipeRefusesWithTheAvailableNames` in `internal/shedcli/seed_test.go`; it now runs `bogus-recipe` and `loom` as subtests and asserts `unknown recipe` and every listed name.
- **F3, F4, F6** `826623e3`: `README.md:58`, `docs/overview.md:650`, `tools/sandbox/SANDBOX-CORE-SUITE.md:250` and `contracts/specs/shed-recipe-spec.md:112-113` say tapestry.
  The affected-set search found one more site of the same default-recipe statement, `docs/overview.md:517` (`loom` when the run has no seed); the code default is `shedrun.RecipeTapestry`, so it is changed too.
  No other spaced seed form with the old value exists under `tools/sandbox`.
- **F5** `8c194b1b`: the comment at `internal/battencli/arm.go:203` names `"tapestry"`.
- **F7** `de970509`: recipe values and recipe-keyed map keys in the listed tests say tapestry; every config-module `loom` key stays.
  Three sites needed more than the key rename: the `name == "loom"` branch in `decisionswitches_test.go`, the `no loom block` text in `runprofile_test.go`, and the legacy fixture `"type":"tapestry"` in `board_test.go` that its assertion reads.
- **F8** `4dfcb107`: `internal/lenses/doc.go` lines 1 and 13 and `internal/reviewbudget/doc.go` line 1 say loom for the config module.
- **Self-check follow-ups** `0fb8bee6`:
  `internal/reportengine/timingspans.go` reads a past run's `Loom-Preflight` row as setup (new `retiredLoomPreflightRow`, one row in `loomRowPhases` with its two totals in `timing_test.go`), and `internal/reportengine/doc.go:25` names the tapestry seed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, `internal/reportengine/doc.go:25` still says the loom recipe: fixed as proposed (`0fb8bee6`).
- fix-consistency, LOW, note that the landing handback readers have no `Loom-Preflight` counterpart to their `Merge-In` handling: no defect by the lens's own account; `Preflight` is never a handback stop row, so no change.
  Disputed on `premise false`, listed below.
- fix-proposal, MEDIUM, `internal/boardengine/legacy.go:118` copies a pre-upgrade `type` into the recipe unmapped, so a real legacy `"type":"loom"` loads as the refused name `loom`: deferred, see below.
- fix-proposal, LOW, the `project tapestry` row no longer shows a project refusing the old name: disputed on `premise false`, listed below.
- fix-affected-set, LOW, `internal/reportengine/timingspans.go` reads a past run's `Loom-Preflight` as other: fixed as proposed (`0fb8bee6`).
- fix-affected-set, note on `internal/boardengine/legacy_test.go:256` feeding `"type":"loom"` as legacy data: same deferred item as the MEDIUM above.

## Disputed

- **Self-check fix-consistency, landing handback note**, ground `premise false`: no code reads `Loom-Preflight` from a child status (`internal/landingshed/handback.go:13` and `parentalign.go:24` read stop rows, and `Preflight` is never one), so there is no handling to add.
- **Self-check fix-proposal LOW, `internal/boardengine/task_test.go:187`**, ground `premise false`: the row pins the project recipe refusal text, which holds with `tapestry`; the refusal of `loom` as an unknown recipe is pinned by the F2 rows, so a second `loom` row here would repeat that behaviour.

## Deferred

- **Self-check fix-proposal MEDIUM, legacy `type` → recipe mapping in `internal/boardengine/legacy.go:118`.**
  Mapping `loom` to `tapestry` on load is an alias for the retired recipe name, which the review's F1 fix text rules out for the recipe name ("it adds no alias for the recipe name") and the decision record's D6 treats as refused.
  Whether pre-upgrade board files may carry the old name, and whether the legacy window is long enough to need the mapping, is an operator decision on a real tradeoff.
  The test fixtures `board_test.go:184` and `legacy_test.go:256` stay as they are until that is decided.

No friction note written.
