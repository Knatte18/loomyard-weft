# Drop Webster's inline kick-start pack layout

`webster-drop-inline-pack` · Note · bug, stencil, webster

## Seen

Found by the stencil intent audit (`docs/research/stencil-intent-audit-2026-10-15/findings.md`, F3); line numbers as of `main` on 2026-10-16.

Webster's kick-start pack has three layouts: per card (`kickstart_pack_per_card`, the default in a hub run), a pack file (`kickstart_pack_file`, the standalone fallback) and inline in the fork prompt (both keys false).
The inline layout's fragments are frozen as a measurement baseline (`contracts/stencils/webster/webster-fragment-fresh-read-pack-inline.md:4`, `webster-fragment-locate-pack-inline.md:4`, `webster-fragment-pack-section-inline.md:4`), so `webster-fragment-fresh-read-pack-inline.md:8` still says "one Master read before forking you", which `webster-master-no-code` made untrue for the other layouts.

## Decision

Operator decision (2026-10-16): never put code-location text straight into a prompt.
Inlined text goes stale as the batch's cards change the tree, and it gives the agent no sign of what it has already read from the source.

## Idea

Agent's suggestion: remove the inline layout, its three fragments and the measurement-baseline freeze, so `kickstart_pack_file: false` with `kickstart_pack_per_card: false` no longer selects it; the per-card layout and the pack-file fallback stay.
