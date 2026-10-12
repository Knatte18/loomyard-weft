# Decision record — kickstart-read-all-first

## Goal

With a kick-start pack, Webster's implementer fork reads every card file of its batch and every span the pack lists under every card in its first turn, in parallel, before any edit.
The rule that a file an earlier card of the batch edited is read whole in that card's step 1, and its member found by name, goes away.

## Scope

In:

- The pack-on read texts of the implementer fork prompt: the pack-on FRESH-READ fragment, the pack-on card-read (step 1) fragment and the pack-on locate fragment under `contracts/stencils/webster/`.
- The tests that pin the pack-on wording (`contracts/stencils/claims_test.go`, `internal/websterengine/template_test.go`, `internal/websterengine/beginbatch_test.go`).
- `internal/websterengine/doc.go`'s description of the pack-on step 1 and FRESH-READ texts and of what bounds a stale read.

Out:

- Pack-off text, the recovery prompt and the verify-fix prompt: they keep rendering byte-identically.
- The pack's content, grouping and rendering: the pack stays per card, headed by the card's pointer path, resolved once at begin-batch.
- The Master prompt, the Burler-side pack (`kick-start-pack`), `enclose-kickstart`, and the with/without audit (`kickstart-pack-measure`).
- Go control flow: `renderReadRules` keeps choosing the pack-on or pack-off fragment pair; the body keeps its `card_read` and `fresh_read` markers.

## Decisions

1. **The first-turn rule lives in the pack-on FRESH-READ fragment.**
   It tells the fork that its first turn, before any edit, reads in parallel every card file the card list points at and every span the pack lists under every card, each span with the Read tool's `offset` at its start line and a `limit` reaching its end line.
   A span listed under several cards is read once.
   The fragment keeps its opening: inherited context can be stale because a prior batch's commits may have changed a file, and only the fork's own reads, taken now, are current.
   Rationale: FRESH-READ is the first section of the body, so the fork meets the rule before the per-card loop, and the body keeps its markers, so no Go or marker change is needed.
2. **After the first turn and before any edit: the files no span covers, and the unresolved glyphs.**
   This extends the board entry, which names only card files and spans: it moves the existing per-card reads of uncovered files and unresolved glyphs ahead of the first edit, batch-wide.
   The FRESH-READ fragment tells the fork, before any edit, to read whole every existing file any card's target list or `Uses:` list names that no listed span covers, and to locate every unresolved member glyph of every card with `lyx quarry resolve` or a grep, then read its file whole.
   A `plan:` handle in an unresolved list is never located, since it names a member a card creates.
   Rationale: card files must be read before their plain-path targets and `Uses:` lists are known, so these reads cannot join the first turn; doing them batch-wide keeps the entry's "everything read before any edit".
3. **Pack-on step 1 does not re-read the card file or spans.**
   The pack-on card-read fragment says the card file, read in the first turn, is the whole instruction for that card, and keeps the existing `**Intent:**` fallback to the Card Index in `_lyx/plan/00-overview.md`.
   Rationale: the read already happened; step 1 only names which file governs the card.
4. **The whole-file rule is dropped.**
   No pack-on fragment tells the fork to read a file whole because an earlier card of the batch edited it, nor to find a member by its glyph's member name for that reason.
   From the pack-on FRESH-READ fragment, the sentence calling the turn at each card's start that card's fresh read and the sentence naming step 1's whole-file read both go.
   Rationale: every listed offset is valid in the first turn, since batches run one fork at a time and no edit has happened yet; later edits use the Edit tool's text matching, which does not depend on line numbers.
   Bound: what can now get through is a later card's span held in context from the first turn and changed since by an earlier card's edit or a self-fix.
   An Edit whose `old_string` no longer matches fails loudly, and the card and batch gates, which run every card's tests, catch reasoning over stale content that slips past.
5. **Listed line numbers hold before any edit only.**
   The pack-on locate fragment's opening clause becomes "… and your first turn read every listed span", replacing "step 1 read the card's listed spans".
   Its "Spans are as of the batch's start … find the member by its glyph's member name" sentence becomes one saying the listed lines describe the tree before any edit, so after an edit the fork locates code by its text and never re-reads by a listed line.
   It keeps "do not call `lyx quarry resolve` or grep for a resolved entry", the `plan:` sentence, the unresolved-glyph sentence, the spec pointer, and "the pack is a kick-start, not a complete list, so read further code, callers or whole files whenever the card needs it".
   Rationale: the board entry keeps further reads open; this sentence is the one guard left against a stale offset, and it costs no read.

## Constraints

- The existing pack-off and recovery byte-identity check in `template_test.go` passes unchanged.
- `PATTERN-stencil-ownership`: the texts stay stencils under `contracts/stencils/webster/`, read through `stencilstore` at call time; no stencil is added or removed, and each keeps its leading banner, `.gitattributes` entry and `stencils.go` registry row.
- `PATTERN-producer-pointer-rule`: the fragments keep pointing at `loom-plan-spec.md` for card grammar and never restate it.
- `PATTERN-comment-line-breaks` and the repo's semantic line breaks apply to the `doc.go` and stencil edits.
- `PATTERN-test-economy`: wording pins extend the existing `claims_test.go` and `template_test.go` rows; no new top-level test, so `test-budget.yaml` is not touched.

## Auto-mode assumptions

- The operator's primary ask (read all first) is built, not the alternative (resolve the pack per card with quarry); see the support log.
- The reads of uncovered files and unresolved glyphs are batch-wide, before any edit (Decision 2).
- No cap on the first turn's read count: batch size is bounded by the active batcher profile, and the board entry accepts a larger first turn.

## Open risks

- A large batch makes the first turn wide; a pack with many resolved entries puts all their spans into context before the first edit.
- The `kickstart-pack-measure` audit's "calls before the first edit" metric rises by construction under this design; that audit compares exploration per card or total reads, not calls before the first edit.
- A fork may reason over a span made stale by an earlier card's edit (Decision 4's bound).

## Acceptance criteria

- A pack-on fork prompt tells the fork to read, in its first turn and in parallel before any edit, every card file and every span listed under every card, with `offset` and `limit`, reading a span listed under several cards once.
- A pack-on fork prompt still says inherited context can be stale and only the fork's own current reads count.
- A pack-on fork prompt tells the fork, before any edit, to read whole every existing target or `Uses:` file of any card that no listed span covers, and to locate every unresolved member glyph of every card and read its file whole.
- A pack-on fork prompt says a `plan:` handle in an unresolved list is not located.
- These rules hold for a card naming no member glyph, a card whose member glyphs all failed to resolve, and a batch where quarry was unavailable: every card file is still read in the first turn.
- A pack-on fork prompt's step 1 does not tell the fork to re-read the card file or spans; it names the card file as the card's instruction and keeps the `**Intent:**` fallback.
- No pack-on text says step 1 reads spans or is a card's fresh read, tells the fork to read a file whole because an earlier card edited it, or to find a member by its glyph's member name for that reason.
- The pack-on locate text says listed lines hold before any edit only, and keeps the no-resolve-for-resolved-entries rule, the `plan:` rule, the unresolved-glyph path and the read-further-when-needed sentence.
- A pack-off fork prompt and a recovery prompt render byte-identically to before.
- The pack section still renders one subsection per card under the card's pointer path.
- `doc.go` describes the first-turn read and states Decision 4's bound in place of the member-name rule.

## Notes for the plan writer

- The pack-on card-read fragment fills `1. {{.card_read}}` after the step number, so its continuation lines keep their 3-space indent.
- `template_test.go`'s pack-on case pins only a prefix of the locate fragment's opening sentence; extend it so the stale "step 1 read the card's listed spans" tail cannot survive.
