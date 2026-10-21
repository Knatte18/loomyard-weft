---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/boardengine/rename_test.go:19
    summary: whole-slug mention boundary rules are untested at text start, text end, exact-match text and beside a digit
    fix: In internal/boardengine/rename_test.go, extend the existing newRenameBoard fixture, or add rows to a small table inside TestRename_KeepsEntryRewritesDependentsAndReportsMentions, so the mentions assertion covers a body that is exactly the old slug, one that starts with it, one that ends with it, and entries holding only old2 or 2old as non-mentions. Update the wanted Mentions list to match. Add no new test function.
    origin: lens:code-tests
  - id: F2
    severity: NIT
    class: design
    location: internal/boardcli/verbs_test.go:109
    summary: the rename verb's id addressing is never exercised through the CLI payload
    fix: In stepRename in internal/boardcli/verbs_test.go, make one of the successful renames address the entry by id, for example by reading the id from the get envelope and passing {"id":N,"to":"..."}, so the lookupSelector wiring and the renamePayload exclusive pair are pinned.
    origin: lens:code-tests
  - id: F3
    severity: NIT
    class: consistency
    location: internal/boardengine/rename.go:36
    summary: the Store.Rename doc comment defines its refusals by pointing at Board.Rename instead of stating them
    fix: In internal/boardengine/rename.go, replace "with Board.Rename's refusals" in the Store.Rename doc comment with the refusals themselves, as Board.Rename's comment lists them, so the comment stands alone.
    origin: handler
---

### [LOW] Mention boundary rules lack start, end and digit coverage

**Location:** internal/boardengine/rename_test.go:19
**Issue:** `mentionsSlug` decides a mention by the runes beside the slug. Every fixture mention sits mid-text, and the near-miss entry only holds `old-thing`, `bold` and `old_x`. A broken start or end boundary, or a digit wrongly counted as a non-slug rune, would pass every test. The code itself is correct: at a text edge the decoded neighbour is `RuneError`, which is not a slug rune.

### [NIT] The rename verb's id addressing is untested at the CLI

**Location:** internal/boardcli/verbs_test.go:109
**Issue:** `stepRename` passes only `slug` payloads. The engine test covers an id selector, but no test reaches the payload `id` path of the new verb.

### [NIT] Store.Rename doc depends on another symbol

**Location:** internal/boardengine/rename.go:36
**Issue:** The comment says the refusals are `Board.Rename`'s, so it is not readable without that symbol's comment.

## Rejected

- Slug-or-id addressing goes beyond the entry's `{slug, to}`: the summary names it, it matches every other addressing verb, and removing it would only reduce the verb.
- Self-rename refused rather than a no-op: the summary names it, and the entry gives no contrary rule.
- A claim in the summary that the diff does not bear out: none found; the Not covered section matches the diff.
- Run lock bypass through the rename translation: the translation is gated on `RenameResult`, applies one old-to-new pair, and other writes are judged as before.
