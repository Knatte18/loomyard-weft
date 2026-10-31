```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits. I looked for edits each card's change forces that no card lists.

**BLOCKING — Card 4 (sampler-session-row) and Card 12 (operator-wait-rows): loomshed's import allowlist is not updated.**
- Production code in `internal/loomshed` may only import the paths in `internal/loomshed#loomshedAllowedImports` (`internal/loomshed/seam_enforcement_test.go:27`). `internal/loomshed#TestToldGeometryInvariant_AllowlistOnly` (`:83`) enforces this.
- Card 4's producer imports packages that are not on the list:
  - `internal/samplerengine`, through `samplerengine.UpdateRecord` and `ReadRecord`.
  - `internal/shedtransient`, through `shedtransient.Class` for the transient start failure.
- Card 12's `OperatorWaitRows` imports `internal/pawlengine` for `pawlengine.NameBeamAwait`, which is not on the list either.
- No card lists the allowlist or its test, so each card's own tier0 gate fails.
- Fix: list `internal/loomshed#loomshedAllowedImports` under Edit on Card 4 (adding samplerengine and shedtransient, each with a no-geometry comment) and on Card 12 (adding pawlengine).

**LOW — Card 6 (sampler-recipe): `RecipeRow.OnDone` breaks an existing assertion that the Intent doesn't mention.**
- `internal/loomrecipe#TestRowNames_FollowEachRecipesRowOrder` compares each `Rows()` entry against a literal with no `OnDone` (`internal/loomrecipe/recipe_test.go:279`).
- Once `Rows` fills `OnDone`, every row with an `on_done` fails that comparison in every recipe.
- The card lists the test under Edit, but its Intent says only "a sampler row". The Intent should say `wantRow` gains `OnDone: row.OnDone`.

**LOW — Cards 6 and 7: comments and docs that name the loom recipes as "tapestry or darn" are only partly covered.**
- A sampler run behaves as darn does in each of these. Card 7 covers two specs, the PR-Review stencil comment and `gateslot/doc.go`. It misses:
  - Stencils, which agents read: `contracts/stencils/orch/orch-template-role.md:26` ("tapestry or darn as the board entry's `recipe` says") and `:137`; `contracts/stencils/loom/loom-rubric-code-review.md:1` and `contracts/stencils/stencils.go:97`. Sampler's perch reads this rubric.
  - `contracts/specs/loom-final-summary-spec.md:9`.
  - landingshed: `fixerrun.go:25`, `:300`; `handback.go:1`, `:26`; `deps.go:74`; `doc.go:95`, `:161`; `finalize.go:621`; `parentalign.go:122`.
  - arbiterstop: `doc.go:14`, `:22`; `arbiterstop.go:67`.
  - `internal/sheddriver/doc.go:73`.
  - battenshed: `doc.go:137`; `deps.go:25` (says `lym loom start` serves tapestry and darn); `innerrun_binaryrestep.go:53`.
  - battencli `wire.go`: `:1086`, `:1273`, `:1482`.
  - loomcli: `drivewiring.go:129`, `settledguard.go:2`, `arm.go:408`, `subjects.go:1`, `doc.go:305`.
  - loomrecipe: `finalizerow.go:1`, `:15`; `bouncerrows.go:1`, `:15`; `doc.go:42`.
  - `docs/overview.md`: lines 262, 267, 465, 592, 635, 657, 679 (batten's admitted recipes, which Card 7 changes), 762.

**LOW — Cards 6 and 7: `docs/overview.md` is listed under Extends but would be rewritten.**
- Card 6 lists it under Extends yet says it "adds it to the lines listing the loom recipes". That rewrites existing lines 522, 635 and 762, so it belongs under Edit (rubric: Extends on a ref the card rewrites).
- Card 7 changes batten's admitted-recipe refusal but doesn't list `docs/overview.md`, whose lines 657 and 679 state the old set.

Tool calls I made:
1. Grepped tests and docs for pinned recipe lists, battencli's use of `pawlengine`, and stencil and module lists. My context had no search for pins outside the cards' named members.
2. Grepped non-test Go, md and yaml for "tapestry or darn" and similar recipe-set wording, to find doc sites the vocabulary change makes wrong.
3. Read `loomrecipe/finalizerow.go`, `bouncerrows.go` and `arbiterstop.go`'s eligibility, to check whether the hard-coded tapestry and darn recipe sets misread sampler rows. They don't, since the row names and engines are shared.
4. Grepped for `RecipeRow{` literals, loomshed's seam enforcement, role tables and module-list pins. My context hadn't seen an import allowlist.
5. Read the rest of the loomshed allowlist and `recipe_test.go:270-285`, to confirm the missing imports and the struct comparison.
6. Read the allowlist test's function name, to cite its glyph.
