# Webster pack-on fork reads every card and span first

With a kick-start pack, the implementer fork now reads every card file of its batch and every span the pack lists under every card in its first turn, in parallel, before any edit.
Before, pack-on step 1 read one card's file and spans, edited, then moved to the next card.
The rule that a file an earlier card edited is read whole, with its member found by name, is gone.

Pack-off text, the recovery prompt and the verify-fix prompt render byte-identically to before.
The pack stays per card under the card's pointer path, resolved once at begin-batch.

## Design decisions

- The first-turn rule lives in the pack-on FRESH-READ stencil.
  The fork meets it before the per-card loop, and the body keeps its `card_read` and `fresh_read` markers, so no Go change was needed.
  Each span is read with `offset` at its start line and `limit` reaching its end line; a span listed under several cards is read once.
  The fragment keeps its warning that inherited context can be stale and only the fork's own current reads count.
- After the first turn and before any edit, the fork reads whole every existing file a card's targets or `Uses:` list names that no span covers, and locates every unresolved member glyph (`lyx quarry resolve` or grep) and reads its file whole.
  A `plan:` handle is never located.
  These reads cannot join the first turn, since card files must be read before their targets are known.
- The fork changes a file it has already edited in this batch only with the Edit tool, never by rewriting it with Write from content read before the edit.
- Pack-on step 1 no longer re-reads the card file or spans; it names the card file as the card's instruction and keeps the `**Intent:**` fallback.
- The pack-on locate fragment now says every line number obtained before the first edit, the pack's listed lines and those from quarry or grep, holds before any edit only.
  After an edit the fork locates code by its text.
- Offsets are all valid in the first turn because batches run one fork at a time and no edit has happened yet.
  Later edits use Edit's text matching, which does not depend on line numbers.

Wording pins extend `wordingClaims`, `TestRenderPrompts` and `TestBeginBatch_KickstartPack`; `internal/websterengine/doc.go` describes the first-turn read and the bound below.

## To check by hand

- A large batch makes the first phase wide: context holds every card file, every listed span and every whole file read for uncovered targets and located glyphs.
  There is no cap on the first turn's read count; the active batcher profile bounds batch size.
- A fork may still reason over a span or whole file read before the first edit and changed since by an earlier card's edit or a self-fix.
  An Edit whose `old_string` no longer matches fails loudly, and the Edit-only rule stops a Write from silently dropping an earlier edit; the card and batch gates catch the rest.
- No full fork run on a real batch with a pack was done by hand; the change is covered by the wording pins only.
