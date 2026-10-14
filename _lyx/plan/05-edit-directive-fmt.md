# Card 5 — edit-directive-fmt

**Edit:**
- `contracts/stencils/edit/edit-directive.md`
- `internal/editdirective#TestDirective_RendersTheRuleWithoutBannerOrMarkers`

**Intent:** D6's directive half: the shared edit directive names `lyx loom fmt`, which card 4 created, so a role told never to rewrite files with a script learns the one sanctioned way to fix a Go file's formatting.

Add one line to `contracts/stencils/edit/edit-directive.md`, after the "Reading with `sed -n`, `grep` or `cat` is fine." line: "A Go file's formatting is fixed with `lyx loom fmt <file>`."
The directive still renders through `internal/editdirective` into every spawned role's opening stencil (`PATTERN-edit-directive`), and the verb is not denied by the agents' settings, so naming it keeps `PATTERN-no-denied-recovery`.

Extend the phrase list `TestDirective_RendersTheRuleWithoutBannerOrMarkers` checks the rendered directive for with "`lyx loom fmt <file>`", so the test fails if the line goes missing from the rendered directive.

**ImpactSummary:** one stencil line every spawned role's opening prompt carries, and one asserted phrase in editdirective's test.

**Commit:** 5: Name lyx loom fmt in the edit directive
