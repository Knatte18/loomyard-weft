# Card 8 — edit-directive-fmt

**Edit:**
- `contracts/stencils/edit/edit-directive.md`
- `internal/editdirective#TestDirective_RendersTheRuleWithoutBannerOrMarkers`

**Uses:**
- `plan:internal/loomcli#loomCLI.fmtCmd`

**Intent:** Name `lyx loom fmt` in the shared edit directive (D6), so an agent has a sanctioned way to fix a Go file's formatting that `cmd/lyx/gofmt_test.go` would fail.
Add one line after the `sed -n` line of `edit-directive.md`: "A Go file's formatting is fixed with `lyx loom fmt <file>`."
The stencil stays marker-free and still renders through `editdirective` (`PATTERN-edit-directive`).
`TestDirective_RendersTheRuleWithoutBannerOrMarkers` adds "`lyx loom fmt`" to the phrases the rendered directive must contain, which fails without the line.

**ImpactSummary:** One added line in the shared edit directive every spawned role renders, and one asserted phrase in its test.
