---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lym/plan/22-raddle-file-producer.md:23
    summary: Cards 19 and 22 import raddleparser and boardengine into production loomshed, but loomshed's import allowlist test admits neither and no card edits it.
    fix: >-
      In card 19, add an `**Edit:**` section (between `**Create:**` and `**Extends:**`) listing
      `internal/loomshed#loomshedAllowedImports`, and add one Intent sentence saying the allowlist in
      `internal/loomshed/seam_enforcement_test.go` gains `github.com/Knatte18/loomyard/internal/raddleparser`
      with a comment line (the proposal parser and its finding type, told paths, resolves no geometry).
      In card 22, add an `**Edit:**` section (between `**Create:**` and `**Uses:**`) listing the same symbol,
      and one Intent sentence saying the allowlist gains `github.com/Knatte18/loomyard/internal/boardengine`
      with a comment line (the filing type and the re-check error the producer's seams carry, no geometry
      resolved through it). boardengine's dependencies hold no loomshed import, so there is no cycle.
      Verified with `lym loom validate-plan --overlay`, combined with the F2 fix: no blocking finding the baseline lacks.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lym/plan/25-pawl-raddle-handoff.md:43
    summary: Card 25 says pawlDeps fills the new Paths.HandOffMarker, but the only filler of pawlengine.Paths is hubgeom.PawlPaths, which no card edits.
    fix: >-
      In card 25, add under `**Create:**` `plan:internal/pawlengine#HandOffMarkerName` -> `const HandOffMarkerName = "handoff-sent"`
      (the file name, beside `AwaitMarkerName` in `internal/pawlengine/landrequest.go`); add under `**Edit:**`
      `internal/hubgeom#PawlPaths` and `internal/hubgeom#TestDobbyGeometry`; add under `**Uses:**` `internal/hubgeom/doc.go`
      (the `doc-go-unlisted` check demands it once a hubgeom non-test file is a target).
      Rewrite the Intent so `hubgeom.PawlPaths` sets `HandOffMarker` to `HandOffMarkerName` under the beam's `.lym/beam/`
      directory beside the land request, with `pawlDeps`, which already takes `Paths: hubgeom.PawlPaths(location)`, filling nothing itself;
      say `TestDobbyGeometry`'s directory assertion over the pawl paths gains the marker, and that `TestWire_BeamRecipe`'s
      existing comparison of `c.env.Pawl.Paths` against `hubgeom.PawlPaths` therefore pins the value.
      Verified with `lym loom validate-plan --overlay`, combined with the F1 fix: no blocking finding the baseline lacks.
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lym/plan/21-raddle-write-producer.md:42
    summary: RaddleTable passes return_note as a table value that is empty on a first session but names no Optional entry, so seatengine Table.Validate refuses the first Raddle-Write session.
    fix: >-
      In card 21's `RaddleTable` paragraph, add that the table sets `Optional: []string{"return_note"}`, the way `DiscussionTable`
      lists its optional directives, because `seatengine.Table.Validate` refuses a blank table value not named in `Optional`
      and `stencil.FillWith` refuses an empty marker not named there. Extend `TestRaddleTable` with a first-session row asserting the table
      passes `Validate` with a blank `return_note`, and the return row asserting the filled value. No target change; `internal/seatengine#Table` is already under Uses.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lym/plan/12-raddleparser.md:34
    summary: Parse must refuse unknown keys with encoding/json alone, which in practice means DisallowUnknownFields, a call cmd/lym/tolerantrecords_test.go fails on in any production file; the card names no other mechanism.
    fix: >-
      In card 12's Intent, replace "refusing unknown keys" with the mechanism: `Parse` decodes each object into `json.RawMessage` maps
      (or decodes twice) and compares its keys against the spec's key lists, naming the unknown key in the error, and never calls
      `Decoder.DisallowUnknownFields`, which `PATTERN-tolerant-records` (test) bans and which its background reserves strictness-by-other-means for
      an agent-written contract file such as `proposal.yaml`. Keep the `TestParse` unknown-key row.
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lym/plan/23-raddle-recipe.md:94
    summary: Card 23 arms the recipe in shedcli's recipes table like motif, whose map keys are string literals, and never says the new key is shedrun.RecipeRaddle (PATTERN-named-constants).
    fix: >-
      In card 23's Intent, where `shedcli`'s `recipes` is armed, state that the new map key is `shedrun.RecipeRaddle`, never the literal `"raddle"`;
      `internal/shedrun#RecipeRaddle` is already under Uses. Say the same for each other place the card writes the recipe's name in
      code (`embeddedRecipes`, `RecipeEngines`, `childRecipeSupported`, `unsupportedChildRecipeRefusal`): each takes `shedrun.RecipeRaddle`.
      Test files may spell the YAML key as the profile template does.
  - id: F6
    severity: MEDIUM
    class: design
    location: _lym/plan/22-raddle-file-producer.md:30
    summary: Card 22 prescribes a Stuck-with-Halt branch and a goto way forward for a SplicePart error that its own text says no proposal passing D6 reaches, because Check with filing true runs the unknown-id checks just before step 2.
    fix: >-
      In card 22, delete the paragraph at line 30 that prescribes the Stuck-with-Halt branch for a `SplicePart` error, and the
      `TestRaddleFile` row "a `SplicePart` error halts naming the goto". A `SplicePart` error is returned as the step's error, unwrapped
      beyond naming the part. The inputs that could reach it, a decision id, criterion or risk bullet, or PATTERN id the record lacks, are
      rejected earlier in the same Call by card 12's `decision-unknown`, `criterion-unknown`, `risk-unknown` and `pattern-unknown`
      through `CheckRaddleProposal(check, true)`, which reads the same decision record. Also drop the matching "revise through goto" clause
      from the Intent.
  - id: F7
    severity: LOW
    class: scope
    location: _lym/plan/22-raddle-file-producer.md:31
    summary: Card 22 says where the project's type label comes from but not where a part's type label and the module labels of either entry come from, though validateLabels refuses a task or project with no type label.
    fix: >-
      In card 22's step 3 sentence, add that each part's `RaddleEntry` carries the part's own proposal `type` as its type label,
      the project's carries the proposal's project `type`, and both carry the module labels the `Entry` seam returns (the Sketch's non-type labels),
      never the `sketch` label. Add to `TestRaddleFile`'s `tasks` row an assertion over the `RaddleFiling` its recording `File` seam receives:
      each part's labels equal its type plus the Sketch's module labels. Text-only; no target change.
  - id: F8
    severity: LOW
    class: consistency
    location: _lym/plan/21-raddle-write-producer.md:34
    summary: The stencil rule worded as ask the operator nothing until the proposal passes would fail TestStencils_NoUnboundedAskOperator, since its sentence holds a banned phrase and no bound word.
    fix: >-
      In card 21's Intent, say the stencil states this rule in a sentence that avoids the phrase "ask the operator" or holds a bound word of
      `askOperatorBounds` ("never", "do not", "before"), for example "Design nothing, and put no question to the operator, before the proposal passes D6's checks."
      The interactive-mode contact with the operator stays in the Go-filled `{{.mode_rules}}` block, as `PATTERN-parent-directive`'s background places it.
---

### [MEDIUM] loomshed's import allowlist is not extended for raddleparser and boardengine

**Location:** `_lym/plan/22-raddle-file-producer.md:23`, `_lym/plan/19-raddle-checks-and-state.md:9`
**Issue:** `internal/loomshed/seam_enforcement_test.go` pins `loomshedAllowedImports` with `scankit.AssertImportAllowlistNoStale`, a tier0 test that fails on any production import outside the list.
Card 19's `CheckRaddleProposal` returns `[]raddleparser.Finding` and card 22's `RaddleFileDeps` carries `File(boardengine.RaddleFiling)` and matches `boardengine.ErrRaddleRecheck`; today loomshed imports neither package, and no card lists the allowlist symbol as a target.
Cards 19 and 22 would fail their own card gate, and every later card's gate, until the list is edited.

### [MEDIUM] hubgeom.PawlPaths, the only filler of pawlengine.Paths, is not a target of card 25

**Location:** `_lym/plan/25-pawl-raddle-handoff.md:43`
**Issue:** `loomcli.pawlDeps` takes `Paths: hubgeom.PawlPaths(location)` (`internal/loomcli/pawlwiring.go:51`), and `PawlPaths` (`internal/hubgeom/dobbygeom.go:74`) builds the four marker and request paths under `.lym/beam/`.
Card 25 adds `Paths.HandOffMarker` and says `pawlDeps` fills it; `beamUpEntry` then requires it, so an unfilled path refuses every BeamUp, and a path filled in `pawlDeps` alone breaks the existing `TestWire_BeamRecipe` comparison of `c.env.Pawl.Paths` with `hubgeom.PawlPaths(loc)`.
The marker's file name has no declared constant either.

### [MEDIUM] RaddleTable's blank first-session return_note is not named Optional

**Location:** `_lym/plan/21-raddle-write-producer.md:42`
**Issue:** `seatengine.Table.Validate` refuses a blank table value unless `Table.Optional` names it (`internal/seatengine/table.go`, the `requiredValues` rule), and `stencil.FillWith` refuses an empty top-level marker the same way.
Card 21 gives `return_note` as "empty on a first session" and the stencil renders it, but the card never sets `Optional`, which `DiscussionTable` does for its optional markers.
As written the first Raddle-Write session of every run fails table validation.

### [MEDIUM] Parse's strictness has no mechanism that passes the tolerant-records scan

**Location:** `_lym/plan/12-raddleparser.md:34`
**Issue:** The card decodes with `encoding/json` "refusing unknown keys".
`PATTERN-tolerant-records` (test) makes `cmd/lym/tolerantrecords_test.go` fail on any production call to `DisallowUnknownFields`, the one built-in way to do that.
The pattern's background allows an agent-written contract file to stay strict, provided strictness is validated some other way, and the card names none.

### [MEDIUM] shedcli's recipe table key is left to be copied as a literal

**Location:** `_lym/plan/23-raddle-recipe.md:94`
**Issue:** `internal/shedcli/table.go` keys `recipes` by string literals (`"motif"`, `"darn"`, and so on), while every other recipe-name site uses a `shedrun.Recipe*` constant.
Card 23 says shedcli "arms it like motif" and lists `shedrun.RecipeRaddle` under Uses without saying the table key takes it, so the literal is the likely result, a repeated literal against `PATTERN-named-constants`.
Cards 13 and 24 do name the constant where they compare the recipe.

### [MEDIUM] A prescribed SplicePart halt has no input that survives D6

**Location:** `_lym/plan/22-raddle-file-producer.md:30`
**Issue:** The card prescribes a `Stuck` with `Halt` and a `lym loom goto Raddle-Write` way forward for a `SplicePart` error, and admits the error is one "D6's `decision-unknown` and `pattern-unknown` checks leave for no proposal they pass".
`SplicePart` (card 11) errors only on a decision id, bullet or PATTERN id the record lacks, and card 12's `decision-unknown`, `criterion-unknown`, `risk-unknown` and `pattern-unknown` reject each of those, run by `Check(filing=true)` over the same record before step 2.
The only reaching input is a test with a faked `Check` seam, which the rubric's reachability rule does not accept.
Round 1's fixer added those two checks and kept the halt beside them; the checks answer round 1's gap, so the halt branch is what is left to cut, and this finding disputes none of that round's fixes.

### [LOW] Label sources of the filed parts are not stated

**Location:** `_lym/plan/22-raddle-file-producer.md:31`
**Issue:** `validateLabels` (`internal/boardengine/labels.go`) refuses a task or project with no type label.
Card 22 states the project's type label comes from the proposal's project `type`, card 10 pins each part's `type`, and card 16 lists labels on `RaddleEntry`, but no card says the part's `RaddleEntry` takes its type label from the part or that either entry takes the module labels from the `Entry` seam.
No row asserts a part's labels.

### [LOW] The stencil's no-question rule is worded against a scan

**Location:** `_lym/plan/21-raddle-write-producer.md:34`
**Issue:** `contracts/stencils/parentdirective_test.go` fails any non-orch stencil sentence holding "ask the operator" and none of "do not", "don't", "never", "must not", "may not" or "before".
The card paraphrases D4 as "design nothing and ask the operator nothing until the proposal passes D6's checks", which holds the phrase and no bound word; the writer following the paraphrase meets a failing tier0 guard.

## Rejected

- Named constant for `raddle` (focus 1), other than F5: card 13 declares `shedrun.RecipeRaddle` and names every boardengine comparison against it, card 24 lists it under Uses, and cards 8, 20, 22 and 23's `finishesAsSketch` and `arbiterRecipeDescription` choose by a recipe's rows. The role constant `raddleRole`, the directory constant `lymdirs.RaddleDirName` and the loom key are separate concepts with the same spelling, as `discussionRole` and the discussion directory are.
- Reachability of the prescribed refusals (focus 2), other than F6: card 13's two promotion refusals, card 14's three slug refusals, card 24's two start refusals, card 20's verb refusals and card 22's hash, re-check and returns-budget returns each name an input or a test row that reaches them, and the upstream label rule admits card 24's by-hand raddle task. Card 13's and card 14's stated bounds match the checks that run (card 24 judges only a raddle-recipe entry, the replayer skips both rules).
- Labels the board requires (focus 3), other than F7: the decision record's project `type` entry is carried by cards 10, 12, 16 and 22, `validateLabels` needs one type label on a task or project and the plan supplies it for both, and card 13 keeps `sketch` off a filed project and any non-raddle task.
- Parent and edit directives (focus 4), other than F8: card 21 puts `{{.parent_directive}}` and `{{.edit_directive}}` in the stencil and maps the `raddle` role in `roleOpeningStencils`, so both opening-stencil scans reach it; both markers are in the seat engine's reserved list, so `RaddleTable` rightly supplies neither value. The spec's family path follows `stencilstore.RelPath`, so `{{.specs_dir}}/raddle/raddle-proposal-spec.md` matches the flat source file card 10 creates.
- Card order, member glyphs and re-sign arrows, granularity of card 23, verify lines and the decision record's added PATTERN trims were each read and raised nothing; the baseline `lym loom validate-plan --overlay '{}'` reports informational findings only.
