<!-- This is burler's fix-check rubric: the severity vocabulary and finding rules the fork lenses of a twill fixer's self-check work under.
     The fixer reads it from its deployed path before it spawns the forks, so every fork inherits it; internal/burlerengine's selfCheckBlock (prompt.go) names that path in the self-check part.
     It declares no marker.
lyx-stencil: sha256=c3c9542d7635e7fdd1f158fb91c85f1ed94f5a39a772e54145c6a694bfb6067e build=112cccb8f4e09be50a700c9346b294b79f649e17 time=2026-10-16T14:39:17Z -->
# Fix-check rubric

This rubric is for checking a fix you just made to a reviewed artifact, never for reviewing the artifact afresh.
The subject is the change set: every sentence, decision, card, row or code line this round's fixer added, removed or rewrote, and every place that change reaches.
Text the fix did not touch and does not reach is out of scope, however wrong it looks.

## Severity

- `BLOCKING`: the fix made the artifact wrong where it was right, or left a stated premise of a finding unresolved in a way that ships a wrong behaviour.
- `MEDIUM`: the fix contradicts a sibling, states a bound or behaviour the code does not hold, or names a step that cannot be carried out.
- `LOW`: the fix is right but leaves a sibling, a criterion or a cross-reference stale in a way a reader would notice.

## What counts as a finding

- A finding names the changed text it concerns (file and section, decision, card or line), what is wrong, and the evidence: a quote from the artifact, a `path:line` or glyph in the code, a `PATTERN` entry, or a test.
- A finding that only repeats a review finding the fixer already fixed is not a finding.
- A finding that proposes a further improvement the review never asked for, and that the fix did not make worse, is not a finding.
