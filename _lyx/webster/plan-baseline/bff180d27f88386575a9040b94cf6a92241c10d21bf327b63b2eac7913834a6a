# Card 12 — plan-stencil-and-rubric

**Prosa:**
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`

**Edit:**
- `contracts/stencils/claims_test.go`

**Intent:**
Push plans onto member glyphs (D6), now that every check the stencil describes exists.

`loom-template-plan.md`:
- The rules state that `**Edit:**` lists the member glyphs the card changes, and a file path only for a file with no symbol to name: a test file the card adds tests to, a non-Go file, or a whole file the card creates or deletes.
  For listing a file or package beside a member of it on one card, the stencil points at `redundant-package-target` and `redundant-file-target` in `{{.specs_dir}}/loom/loom-plan-spec.md` rather than restating them (`PATTERN-producer-pointer-rule`).
- Every new member a card creates is declared as a `plan:` handle, whether or not a later card references it; the "used by at least one later card" rule and `handle-unreferenced` leave the text, while `handle-dangling` and `handle-collision` stay.
- A card that changes a member's parameters, results, type parameters or receiver writes that `**Edit:**` bullet with the re-sign arrow, `` `<glyph>` -> `<new declaration head>` ``, the head written like a Create head; an interface method or a struct field takes no arrow, and the card's Intent states its change.
- A card that deletes or re-signs a member lists every caller of it: the file, or the member glyph whose body holds the call, on the same card for a re-sign and on the same or an earlier card for a delete; the stencil points at `caller-uncovered` in the spec for what counts.
- The minimal skeleton's card lists member glyphs under `**Edit:**` (`internal/boardcli#newListCmd`, and `internal/boardcli#Row.Render` with an arrow head), plus a test file path outside the citation prefixes (`path/to/list_test.go`), so `TestStencils_NoBareCrossRepoCitations` needs no new allowlist entry.
- The implementation-card paragraph says the implementation's member glyphs go under `**Edit:**` beside the existing test file that covers its surface.

`loom-rubric-plan-review.md` gains two "Also flag" bullets:
- an `Edit` card whose targets are only Go file paths of non-test files while its Intent names specific symbols; the finding names the member glyphs to list instead;
- an `Edit` member glyph without the re-sign arrow while the card's Intent changes that member's parameters, results, type parameters or receiver; the finding names the arrow to add, and never fires for an interface method or a struct field, which take no arrow.

`claims_test.go` pins the new stencil text (the member-glyph `Edit:` rule, the re-sign arrow, the caller obligation, the skeleton's arrow bullet) and the two rubric findings, and pins that `handle-unreferenced` and the later-card rule are gone from the stencil.

**ImpactSummary:** Prompt and rubric text plus their claim pins; the next Plan-Write and Plan-Review sessions read the new rules once deployed, and no Go behavior changes.
