---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/01-pack-on-read-all-first.md (Intent, the internal/websterengine/doc.go paragraph)
    summary: >-
      The card calls the doc.go span it replaces three sentences, but the span from the step-1 sentence through the member-name-rule sentence is four lines of internal/websterengine/doc.go (156-159), so the implementer cannot tell whether the That-bounds-the-fork's-reading sentence on line 158 goes or stays.
    fix: >-
      In card 01's Intent, doc.go paragraph, change "The three sentences from" to "The four sentences from", so the replaced span is With-a-pack-step-1 (doc.go:156), FRESH-READ-then-names (157), That-bounds-the-fork's-reading (158) and What-bounds-that (159).
  - id: F2
    severity: NIT
    class: consistency
    location: _lyx/plan/01-pack-on-read-all-first.md (Intent, the webster-fragment-locate-pack.md paragraph)
    summary: >-
      The new line-number line of the locate fragment is given as single-backtick inline code that itself holds backticks around lyx quarry resolve, so its exact bytes are ambiguous, while the fresh-read and card-read texts are given in fenced text blocks.
    fix: >-
      In card 01's Intent, webster-fragment-locate-pack.md paragraph, give the two replacement lines (the new first line and the new line-number line) in a fenced text block, each with its 3-space indent, writing lyx quarry resolve wrapped in backticks inside the line exactly as the stencil is to hold it.
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/01-pack-on-read-all-first.md (Intent, the TestRenderPrompts multipart row bullet)
    summary: >-
      After the edit the multipart row keeps needles another needle of the same row subsumes - the existing contains prefix The kick-start pack below lists, under each card's path, where each member glyph on that card is sits inside the added full new locate first line, and the added contains needle Your first turn, before any edit, reads in parallel every card file repeats what the added inOrder needle Your first turn, before any edit already asserts.
    fix: >-
      In card 01's Intent, multipart row bullet, also drop the contains prefix needle The kick-start pack below lists, under each card's path, where each member glyph on that card is (the full new first line replaces it), and either drop the added contains needle Your first turn, before any edit, reads in parallel every card file or make the added inOrder needle that full phrase instead of Your first turn, before any edit, keeping one of the two.
---

Card 01 carries every Decision and Constraint of the decision record, and the scope it adds is licensed by it.

- Decision 1: the new FRESH-READ text has the first-turn parallel read of every card file and every listed span with `offset`/`limit`, a span listed under several cards read once, and the kept opening.
- Decision 2: it has the whole-file read of uncovered existing target and `Uses:` files and the locate-then-read-whole of unresolved member glyphs, both before any edit, the never-located `plan:` handle and the Edit-only rule.
- Decision 3: the card-read text names the card file as the instruction read in the first turn, keeps the `**Intent:**` fallback and keeps the 3-space continuation indent.
- Decision 4: the four old FRESH-READ sentences and the card-read whole-file and member-name sentences go.
- Decision 5: the two locate lines are replaced and the rest stays byte-identical.
- Constraints: the nil-pack fork and recovery byte-identity rows stay unedited, no stencil is added or removed, the locate fragment keeps the `{{.specs_dir}}` spec pointer, `doc.go` keeps one sentence per line, and no top-level test is added.

The `wantNone` additions pin the acceptance criterion that no pack-on text keeps the step-1 or earlier-card wording, the added "is not necessarily the version on disk" claim pins the criterion that the opening stays, and the `TestRenderPrompts` and `TestBeginBatch_KickstartPack` row edits pin the criterion's three cases and the Notes' locate-prefix point.
All of it is licensed.

Attack surface: the card's Intent states, for `doc.go`, Decision 4's bound in its own words: what can now get through, the loud `old_string` mismatch, the Edit-only rule and the card and batch gates.
Decision 2's Edit-only rule is a new guard rather than a removed one, and Decision 4's bound covers it.
No defect.

Verify coverage: `contracts/stencils` has no build-tagged tests, and `internal/websterengine`'s `integration` tests run under `go test -tags integration`.
The `tmux` and `llm` vets are compile-only, as the template allows, and the section follows the template's order.
No defect.

Member glyphs: every glyph resolves through `lyx quarry resolve` (`contracts/stencils#wordingClaims` at `claims_test.go:107-857`, both test functions in `template_test.go` and `beginbatch_test.go`, `renderReadRules` at `render.go:154-170`, `prePackFreshRead` at `template_test.go:593-599`).
The stencil `.md` files and `doc.go`, which holds only the package comment, are rightly file targets.
`packStepOne` is a local constant inside `TestBeginBatch_KickstartPack`, so the function glyph is the right target.
No signature changes, so no re-sign arrow is needed or present.

Test economy: the card extends existing `wordingClaims` rows and existing `TestRenderPrompts` and `TestBeginBatch_KickstartPack` cases only.
It adds no test function, and `test-budget.yaml` stays untouched.

`lens-review.md` is `planparser.LensReviewFileName`, a known non-card file the plan writer records its lens review in, and validation tolerates it, so it is no defect.

Further readers of the old wording: a repo grep finds only the three stencils, `claims_test.go`, `template_test.go`, `beginbatch_test.go` and `doc.go`, which are the card's targets, plus a quote in the research measurement report `docs/research/lens-fan-lab-phase1.md`, which is a point-in-time record.
The ImpactSummary holds.

### [LOW] doc.go replacement span is miscounted

**Location:** _lyx/plan/01-pack-on-read-all-first.md (Intent, the internal/websterengine/doc.go paragraph)
**Issue:** The card says "The three sentences from 'With a pack, step 1 reads the card file and its listed spans …' through 'What bounds that is the member-name rule …'", but `internal/websterengine/doc.go:156-159` holds four sentences in that span, the third being "That bounds the fork's reading: it is no longer told to re-read whole a file that holds a listed span …".
The count and the range disagree, so the implementer has to guess whether line 158 survives.

### [NIT] Locate replacement line is given as ambiguous inline code

**Location:** _lyx/plan/01-pack-on-read-all-first.md (Intent, the webster-fragment-locate-pack.md paragraph)
**Issue:** The line-number replacement is written as `` `   Every line number … a `lyx quarry resolve` call …` ``, and the nested backticks end the inline span early, so the stencil's exact bytes, which the new `wordingClaims` needle and the rendered-prompt needle both pin, have to be inferred.
The other two fragments' new texts are given in fenced blocks.

### [NIT] Multipart render row keeps subsumed needles

**Location:** _lyx/plan/01-pack-on-read-all-first.md (Intent, the TestRenderPrompts multipart row bullet)
**Issue:** The row keeps the existing `contains` prefix "The kick-start pack below lists, under each card's path, where each member glyph on that card is" while it gains the full new first line that contains it.
It also gains both a `contains` needle "Your first turn, before any edit, reads in parallel every card file" and an `inOrder` needle "Your first turn, before any edit", and the `inOrder` needle already asserts presence.
Each pair asserts one fact twice.
