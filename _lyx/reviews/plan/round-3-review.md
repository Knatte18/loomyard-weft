---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/19-arbiter-amendment-target.md:3
    summary: Card 19 changes the arbiter's observable behavior but never edits the Arbiter section of internal/loomcli/doc.go, which no other card updates for it
    fix: In card 19, add `- \`internal/loomcli/doc.go\`` to the `**Edit:**` list, after `internal/loomcli#arbiterFixture.deps` and before `contracts/stencils/loom/loom-template-arbiter.md`. In its `**Intent:**`, after the paragraph on `loom-template-arbiter.md` and before the "What it lets through, and the bound" paragraph, add this line - `internal/loomcli/doc.go`'s Arbiter section gains a paragraph - the prompt carries `amendment_target`, which is the loom recipe's key once the current generation's plan is approved and `none` otherwise, read at each spawn so a drive that began before approval still routes; the arbiter may then record a plan amendment and name that target as its goto. Checked with `lyx loom validate-plan --overlay` over the full card 19 text with both changes applied; it reports no blocking finding the `{}` baseline does not.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/24-planparser-target-holds.md:9
    summary: Card 24's file rule does not name the sanctioned path reading, so an implementer could compare a self glyph by string, and card 1 names it for the same job
    fix: In card 24, add `- \`internal/planparser#PathEntryPath\`` and `- \`internal/planparser#HandleBody\`` to the `**Uses:**` list, after `internal/planparser#ParseMintedCSharpID`. In its `**Intent:**`, directly after the sentence ending "or the target is `file` as a path or as its file self glyph.", add this sentence - The file rule reads a target through `PathEntryPath` and compares the path it returns with `file`, and a `plan:` handle's body through `HandleBody`; it never builds a self glyph from `file` or compares a self glyph by string. Checked with `lyx loom validate-plan --overlay` over the full card 24 text with both changes applied; it reports no blocking finding the `{}` baseline does not.
---

### [MEDIUM] Card 19 leaves the Arbiter section of `internal/loomcli/doc.go` stale

**Location:** `_lyx/plan/19-arbiter-amendment-target.md:3`

**Issue:** The decision record's scope says docs land in the same commits, naming "the package `doc.go` files the changes touch", and CLAUDE.md requires a task that changes observable behavior to update the module doc in the same commit.
Card 19 changes what the arbiter may do: it gains an `amendment_target` input, may record `--amends-plan` and may name that target as its goto.
`internal/loomcli/doc.go` has an `# Arbiter` section that describes the verb, the stencil's output shape, the told goto guard and the Go driver's seam, and states that the driver's arbiter seam calls the same path.
Card 19's `**Edit:**` list holds no doc file.
Card 13 edits the same `doc.go`, but its Intent documents only the flag, the record, the restore and the commit rules.
Cards 20 and 21 edit no loomcli doc file, and card 21's doc edit is in `internal/websterengine/doc.go`.
So no card updates the Arbiter section, and D6's rule that the card writing the code also writes its docs is unmet for this card.
The card's own order is fine: it follows card 13, so two `Edit` groups on one file are ordered.

### [LOW] Card 24's file rule does not name how a self glyph is read as a path

**Location:** `_lyx/plan/24-planparser-target-holds.md:9`

**Issue:** The card pins the member comparison well: it reads both glyphs through `ParseRef` and `ParseMintedCSharpID` and compares unit, owner chain and name as `columnsOverlap` does, never by string, which satisfies `PATTERN-glyph-conversion-chokepoint` for that rule.
The last clause, "the target is `file` as a path or as its file self glyph", and the sentence on a `plan:` handle whose body is a self glyph say nothing on how a self glyph becomes a path.
The chokepoint background file allows only `Glyph.UnitPath` and `planparser.PathEntryPath` for that reading and `glyph.Self` for the reverse, and forbids a `#`-trimming suffix operation over a glyph value.
Card 1 names `PathEntryPath` for the same job, and card 24's `**Uses:**` lists neither `PathEntryPath` nor `HandleBody`.
An implementer following the card text alone can write `target == file+"#"` or a trimmed comparison, which `PATTERN-glyph-conversion-chokepoint` forbids and no test catches, since the rows pass either way.

## Rejected

- Card 19 reads the approval at wiring time: it does not.
  `arbiterDeps.AmendmentTarget` is a `func() string` that `settleFresh` calls after `WriteArbiterAttempt` and before `arbiterPrompt`, per `internal/loomcli/arbiter.go`, so the Go driver's once-built deps (`goDriverDepsFor` calling `arbiterDepsFor`) pick up an approval made mid-drive.
- Card 19's Edit targets miss a symbol its Intent changes: every changed symbol is listed (`arbiterPromptInput`, `arbiterPrompt`, `arbiterDeps`, `settleFresh`, `arbiterDepsFor`, the stencil and the three test symbols); no signature changes, so no re-sign arrow applies, and the new fields are struct fields, which take none.
- Card 19's Intent states a non-compilable signature: `arbiterAmendmentTarget(recipeTarget string, approved func() (bool, error)) string`, `AmendmentTarget func() string` and the `planApprovedReader(location.AnchorPath())` call agree with card 13's `planApprovedReader(anchorPath string) func() (bool, error)` and with `lyxcwd.Location.AnchorPath()`.
- Card 19 has no row that builds before approval and spawns after it: the `TestArbiterVerb_SpawnPath` row whose `amendmentTarget` answers `Plan-Burler-Review` only once the attempt record exists reads `none` at `f.deps()` and the target at the spawn, so a wiring-time read fails it.
- Card 19's `TestArbiterAmendmentTarget` is a per-symbol test: the approved, unapproved, read-error and empty-recipe branches are reached by no other row, because the spawn row builds its own deps.
- Any later card reads `arbiterDeps.Prompt.AmendmentTarget` as a wiring-time value or calls `arbiterDepsFor` expecting the old shape: a search of the plan finds `arbiterDepsFor` and `arbiterDeps` only in card 19 and in `lens-review.md`, and only cards 13 and 18 mention `planApprovedReader` and `AmendmentTarget` as the symbols card 19 uses, both earlier than card 19.
- Card 19's wiring-guard error from `loomrecipe.AmendmentTarget` breaks an existing caller: `c.recipe` comes from `resolveSeedRecipe`, which holds a seed recipe `shedrun.WriteSeed` validated against `RecipeNames()`, and no test calls `arbiterDepsFor`.
- Card 24 compares glyphs by string prefix: it does not; the unit, owner chain and name comparison, the one-way direction and the `goPackageDir` adjustment match `columnsOverlap` and the chokepoint background file, apart from the file-rule gap in F2.
