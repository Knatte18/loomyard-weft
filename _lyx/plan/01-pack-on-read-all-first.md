# Card 1 — pack-on read all first

**Edit:**
- `contracts/stencils/webster/webster-fragment-fresh-read-pack.md`
- `contracts/stencils/webster/webster-fragment-card-read-pack.md`
- `contracts/stencils/webster/webster-fragment-locate-pack.md`
- `contracts/stencils#wordingClaims`
- `internal/websterengine_test#TestRenderPrompts`
- `internal/websterengine_test#TestBeginBatch_KickstartPack`
- `internal/websterengine/doc.go`

**Uses:**
- `internal/websterengine#renderReadRules`
- `internal/websterengine_test#prePackFreshRead`

**Intent:**
A pack-on implementer fork reads every card file of its batch and every span the pack lists under every card in its first turn, in parallel, before any edit, and the rule that a file an earlier card of the batch edited is read whole in that card's step 1, its member found by name, goes away (Decisions 1–5).
Only the three pack-on fragments change.
The pack-off fragments, `webster-fragment-locate-quarry.md`, the recovery prompt, the verify-fix prompt, the pack section and card subsection stencils, and Go control flow (`renderReadRules` still picks the fragment pair; the body keeps its `card_read` and `fresh_read` markers) stay as they are.
Each stencil keeps its leading banner, its `.gitattributes` entry and its `stencils.go` row; no stencil is added or removed.

`webster-fragment-fresh-read-pack.md` (Decisions 1, 2, 4).
Below the banner, the text becomes exactly these lines:

```text
Inherited context can be stale.
A file you looked at during orientation — your own, or one Master read before forking you — is not necessarily the version on disk right now: a prior batch's own card commits may have changed it since.
Only your own reads, taken now, are current;
content you merely inherited is not.
Your first turn, before any edit, reads in parallel every card file the card list below points at and every span the kick-start pack lists under every card, each span with the Read tool's `offset` at its start line and a `limit` reaching its end line.
A span listed under several cards is read once.
After that turn and still before any edit, read whole every existing file any card's target list or `Uses:` list names that no listed span covers, such as a plain path or a self glyph's file.
Also before any edit, locate every unresolved member glyph of every card with `lyx quarry resolve` or a grep, then read its file whole.
A `plan:` handle in an unresolved list names a member a card creates, so it is never located.
Change a file you have already edited in this batch only with the Edit tool, never by rewriting it whole with the Write tool from content read before that edit.
```

The old sentences "The one turn at each card's start (step 1) is that card's fresh read …", "In the turn right after it …", "An unresolved member glyph's file is read whole in the turn right after …" and "For a file an earlier card of this batch edited, the fresh read is step 1's whole-file read in that turn." go.

`webster-fragment-card-read-pack.md` (Decision 3).
It fills `1. {{.card_read}}` after the step number, so its continuation line keeps the 3-space indent.
Below the banner, the text becomes exactly:

```text
The card file, which your first turn read, is your whole instruction for that card.
   If its `**Intent:**` field is empty, fall back to that card's one-line intent from the Card Index in `_lyx/plan/00-overview.md`, matched by the same NN/slug.
```

The parallel-read, `offset`/`limit`, whole-file and member-name sentences go.

`webster-fragment-locate-pack.md` (Decision 5).
The first line becomes `   The kick-start pack below lists, under each card's path, where each member glyph on that card is, and your first turn read every listed span.`
The "Spans are as of the batch's start …" line becomes `   Every line number you obtained before your first edit, the pack's listed lines and the lines a `lyx quarry resolve` call or grep returned while locating an unresolved glyph, describes the tree before any edit, so after an edit locate code by its text and never read by any such line.`
The "Do not call `lyx quarry resolve` …", "The pack is a kick-start …", `plan:`, unresolved-glyph and card-grammar (`{{.specs_dir}}`) lines stay byte-identical, so the fragment keeps pointing at `loom-plan-spec.md` and restates no grammar (`PATTERN-producer-pointer-rule`).

`contracts/stencils#wordingClaims` rows:
- `webster-fragment-fresh-read-pack.md`: keep the "Inherited context can be stale." and "Only your own reads, taken now, are current;" claims, and add claims that the text still holds "is not necessarily the version on disk right now: a prior batch's own card commits may have changed it since."
  Replace the step-1 and turn-right-after claims with `wantAll` claims on: "Your first turn, before any edit, reads in parallel every card file the card list below points at and every span the kick-start pack lists under every card", "`offset` at its start line", "`limit` reaching its end line", "A span listed under several cards is read once.", "read whole every existing file any card's target list or `Uses:` list names that no listed span covers", "locate every unresolved member glyph of every card with `lyx quarry resolve` or a grep, then read its file whole", "A `plan:` handle in an unresolved list names a member a card creates, so it is never located." and "Change a file you have already edited in this batch only with the Edit tool, never by rewriting it whole with the Write tool from content read before that edit."
  Add a `wantNone` on "(step 1) is that card's fresh read", "step 1's whole-file read" and "an earlier card of this batch edited".
- `webster-fragment-card-read-pack.md`: replace every claim with a `wantAll` on "The card file, which your first turn read, is your whole instruction for that card.\n" and on "   If its `**Intent:**` field is empty, fall back to that card's one-line intent from the Card Index in `_lyx/plan/00-overview.md`, matched by the same NN/slug.", and a `wantNone` on "in parallel", "`offset`", "read that file whole" and "member name".
- `webster-fragment-locate-pack.md`: the opening needle becomes the full new first line, the "Spans are as of …" needle becomes the full new line-number line, the row's `why` drops "reads each card's listed spans in step 1" and "finds a member … by its glyph's member name", and the `wantNone` gains "step 1 read the card's listed spans", "find the member by its glyph's member name" and "an earlier card of this batch edited".
  The resolved-entry, kick-start, `plan:`, unresolved-glyph and `{{.specs_dir}}` claims stay.

`internal/websterengine_test#TestRenderPrompts` rows, each extended rather than added:
- "fork with a resolved multipart pack lists each declaration and locates by the pack": `contains` drops "in parallel, in one turn, before doing anything else", "not the whole files that hold them" and "find the member by its glyph's member name", and gains the full new locate first line (the rendered line, 3-space indent included, so the stale "step 1 read the card's listed spans" tail cannot survive), "describes the tree before any edit, so after an edit locate code by its text", "Your first turn, before any edit, reads in parallel every card file" and "1. The card file, which your first turn read, is your whole instruction for that card.\n   If its `**Intent:**` field is empty".
  "that no listed span covers" and "read further code, callers or whole files whenever the card needs it" stay.
  `notContains` gains "step 1 read the card's listed spans", "find the member by its glyph's member name", "read that file whole" and "is that card's fresh read".
  `inOrder` gains, ahead of its card-pointer bullets, "## The FRESH-READ rule" then "Your first turn, before any edit", so the first-turn rule sits before the per-card loop.
- "fork with a card naming no member glyph renders the empty line" and "fork with a card whose member glyphs none resolved renders none then its unresolved list": each gains "Your first turn, before any edit, reads in parallel every card file" in `contains`, so every card file is still read in the first turn when no member glyph resolves or none is named.
- The nil-pack fork and recovery rows stay unchanged: their byte-identity check against `writePrePackBody` and `prePackFreshRead` is the pack-off and recovery guard, and it must pass without edit.

`internal/websterengine_test#TestBeginBatch_KickstartPack`: the `packStepOne` constant becomes `packFirstTurn` with the value "Your first turn, before any edit, reads in parallel every card file", every use renamed, and the "an unavailable index degrades every entry to unresolved" case gains `packFirstTurn` in `wantPrompt`, so a batch where quarry was unavailable still tells the fork to read every card file in its first turn.
The switch-off and language-none cases keep `packFirstTurn` in `wantNotPrompt`.

`internal/websterengine/doc.go`, section "the kick-start pack": the sentence "With a pack it takes the pack fragment, which sends the fork to the listed spans …" says the pack fragment says the fork's first turn read the listed spans, that every line number obtained before the first edit holds before any edit only, and keeps the quarry or grep path for an unresolved member glyph.
The three sentences from "With a pack, step 1 reads the card file and its listed spans …" through "What bounds that is the member-name rule …" are replaced by: with a pack, FRESH-READ has the fork read every card file and every listed span in its first turn, in parallel, then the target and `Uses:` files no span covers and the located unresolved glyphs' files, all before any edit, and step 1 only names the card file as the card's instruction.
State Decision 4's bound in place of the member-name rule: what can now get through is anything read before the first edit and changed since by an earlier card's edit or a self-fix; an Edit whose `old_string` no longer matches fails loudly, the Edit-only rule keeps a Write built from a pre-edit read from dropping an earlier edit, and the card and batch gates, which run every card's tests, catch reasoning over stale content.
One sentence per line (`PATTERN-comment-line-breaks`).
The `doc.go` criterion is prose no test reads, so review checks it.

No new top-level test, so `cmd/lyx/testdata/test-budget.yaml` is untouched.

**ImpactSummary:** Prompt text only: three pack-on stencils change, the pack-off, recovery and verify-fix prompts render byte-identically, and no Go code outside tests and `doc.go` changes; the wording pins in `contracts/stencils` and `internal/websterengine` are the only readers of the old text.

**Commit:** 1: Webster pack-on fork reads every card and span in its first turn
