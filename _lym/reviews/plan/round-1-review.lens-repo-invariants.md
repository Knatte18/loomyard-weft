```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. I checked each card for reflective seam coverage, new test tier tags, docs it falsifies and whether it can pass its own gate given the cards before it. Four invariants break.

**Per card, each invariant checked**
- Card 1: seams n/a. No new test file. Doc held: the spec is edited and `discussionparser/doc.go` is under Uses. Order held.
- Card 2: seams n/a. Doc held: `shedrun/doc.go` is edited by card 9, and no doc names the alias. Order held: `TestWire_BeamRecipe` writes no `beam` seed.
- Card 3: seams n/a. New test is a function, no file. Doc held: `shedbuild/doc.go`, `shedrecipe/doc.go` and `shed-recipe-spec.md` are edited. Order held.
- Card 4: seams n/a. Doc held. Order held.
- Card 5: seams n/a. Doc held: `sheddriver/doc.go`, `darnengine/doc.go`, `PATTERN.md` and `docs/overview.md` are edited. `configreg/doc.go:16` describes `KeyCopies` in general terms and stays true. Order held.
- Card 6: seams n/a. Doc held for what I checked. Order held.
- Card 7: seams n/a. Doc held: `loomrecipe/doc.go` is Extends. Order held.
- **Card 8: order broken.** It runs before darn declares `contract: loom` and `verify_source` (card 11).
- **Card 9:** seam held (`PreflightBatcher` is in envkit `nilLegal`, which batten's `NilSeams` skips). Order held. **Doc broken twice:** `reportengine/doc.go` and `pattern/PATTERN-cwd-resolution.md`.
- Card 10: doc held (`loomrecipe/doc.go` edited; the reportengine phase is unchanged). Order held.
- Card 11: seams held (`RecordWrite.*` added to batten's `intentionallyNil`, `FullEnv` edited, `wire` fills it for every recipe). **Doc broken:** `reportengine/doc.go` darn setup line.
- Card 12: doc held (rubric, recipe and overview edited). Order held.
- **Card 13: seam broken** (batten's `DiscussionHistory`). **Doc broken:** `pattern/PATTERN-gate-slot.md`.
- Card 14: held.
- Card 15: seams held (`Darn.CommittedRecord` listed in both cli lists; `FullEnv` fills the new path fields). Doc held.
- Cards 16–20: held for what I checked.
- New test files: none. Every new test is a `plan:` function handle, so no file tier tag is owed.

**Findings**

1. **BLOCKING · design** — card 8 (`08-one-loom-builder.md`), card order. Card 8's `New` refuses any recipe without `contract: loom`, and stamps darn's verify attempts only when `verify_source` is `tree_scope`. But `darn-recipe.yaml` gains both keys only in card 11.
   - Evidence: `contracts/recipes/darn-recipe.yaml:61-66` has no `contract` or `verify_source`; card 11's Intent adds both.
   - From card 8 to card 10, `New("darn", …)` refuses. That fails the darn tests card 8 itself rewrites to pass `shedrun.RecipeDarn` (`internal/loomrecipe/darnrecipe_test.go:38,131,251,309`) and `TestWire_DarnRecipe`.
   - Card 8 cannot pass its own card gate.

2. **MEDIUM · consistency** — card 13 (`13-one-discussion-validator.md`), test seam. Card 13 drops `DiscussionHistory` from envkit's `nilLegal`, but batten's wiring never sets it.
   - Evidence: `internal/testkit/envkit/envkit.go:54`; no `DiscussionHistory` in non-test battencli code.
   - batten's tier0 `TestWire_EverySeamFilled` (`internal/battencli/wire_seams_test.go:184-193`) then fails, because `DiscussionHistory` is not in its `intentionallyNil` (`wire_seams_test.go:37-124`).
   - Card 13 edits `internal/battencli#intentionallyNil` but adds only `Darn.CheckRecord`.

3. **MEDIUM · consistency** — card 13, doc falsified. `pattern/PATTERN-gate-slot.md:28-29` names `loomshed.ValidateLoomDiscussion` as the discussion gate and `validate-discussion` function, and `loomshed.ValidateDarnDiscussion` as Darn-Write's `discussion` gate.
   - Card 13 deletes both validators and Darn-Write's discussion gate, and replaces them with `ValidateDiscussion`.
   - No card lists `pattern/PATTERN-gate-slot.md`.

4. **MEDIUM · consistency** — cards 9 and 11, doc falsified. Both edit `internal/reportengine` code but list `internal/reportengine/doc.go` only under Uses.
   - Card 9 removes Tapestry-Preflight from `loomPhases`, yet `reportengine/doc.go:130` still says tapestry's setup holds Preflight and Tapestry-Preflight.
   - Card 11 adds Record-Write to `darnPhases` setup, yet `reportengine/doc.go:132` lists darn's setup as only Preflight, Drift-Baseline and Drift-Await.

5. **LOW · consistency** — card 9, doc falsified. `pattern/PATTERN-cwd-resolution.md:21` names `loomengine.CheckSeed` as the third resolution tier.
   - Card 9 deletes `CheckSeed` and the seed check, and no card lists that background file.

**Tool calls**
1. grep over `pattern/`, `PATTERN.md`, docs, specs, stencils and package `doc.go` files for removed symbols: the manifest held no doc or `pattern/` background content.
2. grep for those symbols' lines in the docs that matched and that no card edits: needed the exact stale text.
3. grep of `reportengine/doc.go` darn lines and `pattern/` for moved keys and validators: needed the darn phase line and the gate-slot validator lines.
4. grep of `TestWire_BeamRecipe` and the pawl wiring branch: needed to know whether the pawl seam test catches a nil `DiscussionHistory`.
5. Read of `wiring_beam_test.go:55-70`: confirmed the pawl test only checks the `BeamDescribe` and `Pawl` prefixes, so it raises no finding.
