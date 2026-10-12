# Webster kick-start pack: read the card's spans in one turn, per-card packs

`webster-pack-read-set` · Task · enhancement, webster, stencil · done

## Seen (2026-10-12)

An audit of Webster implementer forks found no measurable gain from the kick-start pack (note `kickstart-pack-measure`).
A transcript study of 8 pack-on and 3 pack-off forks (`.scratch/kickstart-study/notes.md` in the hub prime) found why:

- The fork prompt (`contracts/stencils/webster/webster-fragment-locate-pack.md`) says only "Open each listed file and read the listed span before editing."
  Quarry's bench, where the pack worked (`bench/loomyard-eval/cards/07-e1-pack.md`, results `2026-09-06-kickstart`), said "Read all listed spans in parallel, in one turn, before doing anything else."
- The FRESH-READ rule in the same prompt asks the fork to re-read every file the card's target and `Uses:` lists name, which forks take as whole-file reads.
- Forks read the listed files 2-4 per turn, card by card, mostly whole (a 1100-line file for a 180-line span), so the pack only replaces the 1-3 locate turns.
- The pack is batch-wide (11 to 144 entries) while the fork works card by card.
- A multi-line `var` or `const` entry carries its whole initializer as its signature: in one prompt the pack was 76 KB of 92 KB.
- The Master (Merriam) reads no source code and gets no pack, as designed.

## Want

The pack kick-starts a fork; it is no promise that everything is covered (operator, 2026-10-12).
A fork must and may read more on its own when the card needs it, and the source code is never inlined in the prompt (operator).

- The fork reads every listed span for the card in parallel, in one turn, with offset and limit, before anything else, in the bench's wording.
- FRESH-READ re-reads the listed spans, not whole files.
- The pack is grouped per card, so the fork reads card N's spans at card N.
- A `var` or `const` entry shows only its first line.

## Measure after

Compare runs with `kickstart_pack` on and off, as in `kickstart-pack-measure`.
