# Card 5 — edit-directive-fmt-line

**Edit:**
- `contracts/stencils/edit/edit-directive.md`
- `internal/editdirective#TestDirective_RendersTheRuleWithoutBannerOrMarkers`

**Intent:** D6's directive line: the edit directive gains one line, placed after the `Reading with` line: "A Go file's formatting is fixed with `lyx loom fmt <file>`."
It still renders through `internal/editdirective` (`PATTERN-edit-directive`), and it is the directive's only statement of that route, so no other stencil restates it.
`TestDirective_RendersTheRuleWithoutBannerOrMarkers` adds "`lyx loom fmt`" to the substrings the rendered directive must contain, so the rendered edit directive is pinned to name the verb.
The line names a verb card 4 creates, so it lands after it.

**ImpactSummary:** One stencil line every spawned role's opening stencil renders, and one assertion in the directive's render test; no Go code changes.

**Commit:** 5: Name lyx loom fmt in the edit directive
