# Webster fork deviations list mixes in its own target files

`webster-deviations-list` · Note · bug, webster, stencil

## Seen

Recorded in note `glyph-use-audit` (removed 2026-10-15): a Webster fork deviations list mixed in files that probably house its batch own target glyphs, because the implementer stencil asks it to compare against the glyph union and also tells it not to.

## Idea

Give the stencil one rule for what counts as a deviation, matching what `ScopeGuard` checks, and check it again after `kickstart-per-card` lands, since that task rewrites the fork prompt.