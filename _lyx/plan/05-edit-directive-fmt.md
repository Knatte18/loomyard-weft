# Card 5 — edit-directive-fmt

**Edit:**
- `contracts/stencils/edit/edit-directive.md`
- `internal/editdirective#TestDirective_RendersTheRuleWithoutBannerOrMarkers`

**Intent:**
D6's second half: the edit directive names the sanctioned way to fix a Go file's formatting, the `lyx loom fmt` verb card 4 adds.

`contracts/stencils/edit/edit-directive.md` gains one line after the `sed -n` line and before the `.scratch/` line: ``A Go file's formatting is fixed with `lyx loom fmt <file>`.``
The stencil stays marker-free and keeps its leading comment, and it still renders through `internal/editdirective`, per `PATTERN-edit-directive`.
`lyx loom fmt` is not a denied form, so `PATTERN-no-denied-recovery` holds.

`TestDirective_RendersTheRuleWithoutBannerOrMarkers` adds `` `lyx loom fmt <file>` `` to the phrases the rendered directive must contain, so a directive that drops the line fails.

**ImpactSummary:** One line added to the shared edit directive, which every spawned role's opening stencil renders.
