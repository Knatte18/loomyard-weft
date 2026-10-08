---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/websterengine/beginbatch.go:274
    summary: Three edited comment sentences still break mid-phrase on an old column wrap, in beginbatch.go, planglyph/repo.go and planglyph/handle.go, the defect class round 2 F1 raised and fixed only at the sites it named
  - id: F2
    severity: LOW
    class: consistency
    location: internal/planglyph/deleteorder.go:89
    summary: collectEditTargets doc pairs the plural cards with the singular verb is, and editTarget doc at line 81 still lets that card be read as the member glyph or file
  - id: F3
    severity: LOW
    class: consistency
    location: internal/planglyph/doc.go:39
    summary: the edited sentence naming the passes that raise glyph-rejected omits deleteorder.go, whose unreadableAnswerFinding raises it
  - id: F4
    severity: NIT
    class: consistency
    location: internal/planglyph/glyphchain_test.go:488
    summary: TestGlyphChain_CrossCard keeps a want column that no case sets
---

### [BLOCKING] Edited comment sentences still break mid-phrase

**Location:** `internal/websterengine/beginbatch.go:274-275`, `internal/planglyph/repo.go:53-54`, `internal/planglyph/handle.go:357-358`

**Issue:**
`PATTERN-comment-line-breaks` allows a break inside a sentence only after a semicolon or a colon-ended sentence part, or at a comma before an independent clause.
Round 2 F1 applied that rule to a sentence the diff edited and whose leading lines kept an old column wrap, and the fixer rewrapped the sites it named.
Three more sentences have the same shape: the branch's commits rewrote their tail, and their first line still breaks mid-phrase.
`lint-comments` passes because it flags only a break the diff creates.
- `beginbatch.go:274-275`, the comment above `restampFingerprint`: card 5 rewrote the clause after the colon from "the status, Create-inversion and containment passes all run after the" to "the status and Create-inversion passes both run after the rewrite, ...", and the sentence still breaks between "and it" and "then keeps going".
- `planglyph/repo.go:53-54`, the `resolveTargets` doc: card 5 rewrote the consumer list after "nowhere else:", and the sentence still breaks between "one call to" and "(*quarry.Repo).Resolve".
- `planglyph/handle.go:357-358`, the `cardOwnHandles` doc: card 5 cut "and both containment tiers" from the clause after "PG-2):", and the sentence still breaks between "closes the gap a" and "Rename-only card".

I did not raise the sentences whose own words the commits left alone, such as the first sentence of the `ValidateDispatch` doc, the "quarry.Name computes." sentence in the `renameDeclSource` doc, and the first sentence of the `planparser/handle.go` header: they keep an old break and no edit touched them.

**Fix:**
Put each of the three sentences on one line, or break it only at a colon, a semicolon or a comma before an independent clause, and leave untouched neighbouring sentences alone.

### [LOW] collectEditTargets and editTarget doc comments do not parse cleanly

**Location:** `internal/planglyph/deleteorder.go:81` and `internal/planglyph/deleteorder.go:89`

**Issue:**
The `collectEditTargets` doc reads "every Edit-group target of cards that is a member glyph or a file".
The subject "cards" is plural and the verb "is" is singular, so the sentence does not parse.
The `editTarget` doc reads "one Edit-group ref of a card that is a member glyph or a file, with its shape and disk path".
"that is" attaches to the nearest noun, so it says the card is a member glyph or a file.
The type holds a ref that is one of those two shapes, with its shape and disk path.
This is the third focus question for the round, and the round 2 F2 fix did not settle it.

**Fix:**
Attach the clause to the ref.
`editTarget`: "editTarget is one Edit-group ref of a card, a member glyph or a file, with its shape and disk path."
`collectEditTargets`: "...every Edit-group target of cards, limited to member glyphs and files."
Keep one sentence per line.

### [LOW] doc.go names an incomplete set of glyph-rejected passes

**Location:** `internal/planglyph/doc.go:39`

**Issue:**
The commit edited the sentence to read "The passes that raise glyph-rejected are resolve.go, create.go and donecheck.go — one per fail-closed status policy in the package".
`internal/planglyph/deleteorder.go:317` (`unreadableAnswerFinding`) also raises `glyph-rejected`, and `status_enforcement_test.go` lists `deleteorder.go:answerSymbols` as a fail-closed status reader.
The list is false as written.
The old text it replaced said "FOUR passes ... resolve.go, create.go, containment.go and donecheck.go" and had the same omission, but the sentence is now edited text, so the diff owns it.
Card 5 asked for a sentence that "names the passes that raise glyph-rejected without counting them", and a list that misses one pass does not meet that.

**Fix:**
Add `deleteorder.go` to the list, or name the group the sentence needs, "every fail-closed status reader", and point at the allowlist in `status_enforcement_test.go` instead of enumerating files.

### [NIT] TestGlyphChain_CrossCard carries a column no case sets

**Location:** `internal/planglyph/glyphchain_test.go:488`

**Issue:**
The table in `TestGlyphChain_CrossCard` declares `want []findingKey`, and no case sets it.
Every case expects a clean plan, so `assertPlanGate(t, plan, ..., tc.want)` always receives nil.
The column suggests the test can pin a finding that it never pins.

**Fix:**
Drop the field and pass nil, or keep it only if a case needs it.

## Focus verdicts

**Q1, the five named sites.**
No finding.
Every sentence these commits edited sits on one line or breaks at a colon or semicolon:
- `ValidateDispatch` doc in `planglyph.go`: the edited sentence "completed names every card ..." is on one line, and the two paragraph sentences below it are each on one line, with a colon break after "plan:".
- `planparser/handle.go` header: the edited sentences are on their own lines, and the break before "this file never re-implements" follows a semicolon.
- `planparser/handle_test.go` header: one line.
- `planparser/shape.go` ledger doc: two sentences on two lines.
  The `gateRedundantTarget` and `gateResignTarget` comments are each one line.
- `planparser/validate.go` `checkGlyphMalformed` doc: both edited sentences are on one line each.

**Q2, other touched comments.**
F1 names the three sentences, by file, line and sentence start.
A scan of every comment hunk in the diff for an edited block whose preceding line ends mid-sentence found these three plus the unedited-sentence cases F1 excludes.

**Q3, `editTarget`.**
Not settled: see F2.

## Other checks

- Range: no `_lyx/loom/rework` directory exists, so the range is `git merge-base main HEAD..HEAD` with `product.parent` `main`.
- Build, `go vet` on `planglyph` and `planparser`, `go test` for `planglyph`, `planparser`, `websterengine`, `contracts/...` and `cmd/lyx`, and `go run ./cmd/lyx loom lint-comments` all pass.
- Per-card mechanical checks:
  - Card 5's Delete group: a grep for `syntacticContainment`, `resolveContainment`, `gateSyntacticContainment`, `containment-unit-overlap` and `containment-file-overlap` finds no remaining reference outside the plan.
  - Card 6: `declaredHandles` and `handle-unreferenced` are gone from code and spec; the one remaining hit is the stencil claim that pins their absence.
  - The spec's validation-check list is numbered 1 to 30 in the order `validate` runs the checks, and its `row N` cross-references point at the right rows.
- Test economy: the planparser pure-case tests and the planglyph integration rows for `redundant-package-target` overlap on two plan-gate verdicts, but card 7 asks for the cross-gate rows by name, and the integration rows add the before-and-after-binding dimension.
  No other duplicate found.

## Prior rounds

Read after this review was saved.
Round 2 F1 asked for each edited sentence on one line; that class is what F1 above finds at three more sites.
Round 2 F2 is the `editTarget` doc, which F2 above re-opens because the wording the fix took still reads as the card being a member glyph or file.
