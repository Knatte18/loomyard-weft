# Webster kick-start pack: per-card pack, one-turn span reads

An audit of Webster implementer forks found no measurable gain from the kick-start pack.
A transcript study found why: forks read the listed files a few per turn and mostly whole, the FRESH-READ rule sent them back to whole-file reads, the pack was batch-wide while a fork works card by card, and a multi-line `var` or `const` signature carried its whole initializer (one pack was 76 KB of a 92 KB prompt).
This change reshapes the pack and its prompt text so a measurement with `kickstart_pack` on and off can show whether it helps.
The pack stays a kick-start, not a read set: the fork reads further whenever the card needs it, and no source code is inlined in the prompt.

## What changes

- The pack is grouped per card.
  `planindex` gains a per-card wrapper that pairs a card with its own `Pack`, and `KickstartPack` returns the list in the batch's declared card order.
  A ref is listed under every card that names it.
  The batch's distinct refs are still resolved in one quarry call and fanned out to the cards.
- The fork prompt's pack section renders one subsection per card, headed by the same path the card-pointer list shows for that card, so it matches under hub and standalone geometry.
  A card with nothing unresolved shows no unresolved list.
  A card naming no member glyph shows the empty-pack line, reworded to "this card".
  A card whose member glyphs all failed to resolve shows `none` for its entries, then its unresolved list.
  There is no batch-level unresolved list.
- With quarry unavailable, each card lists its selected member glyphs as unresolved with the error and its `plan:` handles with the handle reason; begin-batch still does not refuse.
- Pack-on wording for the implementer body: step 1 and the FRESH-READ rule become markers filled from a pack-on or pack-off fragment.
  - Pack-on step 1 tells the fork to read the card file and every span listed under that card in parallel, in one turn, with the Read tool's offset and limit, before anything else.
    A file an earlier card of the batch edited is read whole in that turn and the member found by its glyph's member name, with the listed signature as a cross-check only.
  - Pack-on FRESH-READ names the listed spans rather than whole files, and has the fork read whole, before any edit, the targets and `Uses:` files that no listed span covers.
  - The pack-on locate fragment adds that the pack is a kick-start and the fork may read callers or whole files as needed.
  - The pack-off fragments are today's text verbatim.
- A `var` or `const` entry keeps only the first line of its signature (kind check in `planglyph`, so `planindex` takes no quarry import); its span is unchanged.
- The `webster: kickstart pack` log line keeps its keys, and `resolved`/`unresolved` still count distinct refs across the batch, so `kickstart-pack-measure` is unaffected.

New prompt text lives in stencils under `contracts/stencils/webster/` with registry rows in `contracts/stencils/stencils.go` and `.gitattributes` entries.
`internal/websterengine/doc.go` documents the per-card pack, the per-mode texts and the `var`/`const` rule.

## Design notes for review

- Pack-off prompts, prompts for a language that opts out of glyphs, and the recovery prompt are meant to render byte-identically to before; `template_test.go` rebuilds the pre-change bytes by substitution to pin this.
- Under the pack-on FRESH-READ text a fork no longer re-reads whole a target or `Uses:` file that holds a listed span, so an edit outside a listed span in such a file is read only when the fork chooses.
  The member-name rule for files edited earlier in the batch, the card gates and the batch gate bound a stale read.
- An unresolved member glyph's file is read whole in the turn after the `lyx quarry resolve` call or grep that locates it.
  A reviewer should check that this turn pin in the pack-on FRESH-READ text is unambiguous.

## Out of scope

The Master prompt, the recovery and verify-fix prompts, the Burler-side `kick-start-pack` and `enclose-kickstart` packs, and the with/without measurement itself (`kickstart-pack-measure`).

## To check by hand

- No with/without measurement was run; the effect on fork reads is unverified.
- The implementer batch's plan named `./internal/websterengine_test` in its gates, a path with no directory of its own; the gates were run on `./internal/websterengine`.
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` were not run as part of this description.
