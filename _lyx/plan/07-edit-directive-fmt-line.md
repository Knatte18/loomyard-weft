# Card 7 — The edit directive names `lyx loom fmt`

**Edit:**
- `contracts/stencils/edit/edit-directive.md`
- `internal/editdirective#TestDirective_RendersTheRuleWithoutBannerOrMarkers`
- `internal/editdirective/doc.go`

**Intent:**
Decision D6's second half, landed after card 6 so the line names a verb that exists.
The directive stencil gains one line, after "Reading with `sed -n`, `grep` or `cat` is fine.":

```
A Go file's formatting is fixed with `lyx loom fmt <file>`.
```

The directive still renders through `internal/editdirective` (`PATTERN-edit-directive`), stays marker-free, and names no denied command (`PATTERN-no-denied-recovery`).
`editdirective/doc.go`'s "What it says" section gains the sentence that a Go file's formatting is fixed with `lyx loom fmt <file>`.
`TestDirective_RendersTheRuleWithoutBannerOrMarkers` adds "`lyx loom fmt <file>`" to its `want` list, so the rendered directive must name the verb.

**ImpactSummary:** Every spawned role's rendered edit directive gains one line naming `lyx loom fmt`; the directive test and package doc follow.

**Commit:** 7: editdirective: name lyx loom fmt for Go formatting
