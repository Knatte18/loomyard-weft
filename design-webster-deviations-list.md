# Webster fork deviations list mixes in its own target files

`webster-deviations-list` · Task · bug, webster, stencil · done

## Seen

Recorded in note `glyph-use-audit` (removed 2026-10-15): a Webster fork deviations list mixed in files that probably house its batch own target glyphs, because the implementer stencil asks it to compare against the glyph union and also tells it not to.

## Idea

Give the stencil one rule for what counts as a deviation, matching what `ScopeGuard` checks, and check it again after `kickstart-per-card` lands, since that task rewrites the fork prompt.

## Operator decision (2026-10-15)

Fix it now, as a small task: `kickstart-per-card` has landed, so check the implementer prompt as it stands today and give it the one deviation rule that matches `ScopeGuard`.