# Card 5 — twilldirective-package

**Create:**
- `internal/twilldirective/doc.go`
- `plan:internal/twilldirective#MarkerName` -> `const MarkerName = "twill_directive"`
- `plan:internal/twilldirective#WriteScopeMarkerName` -> `const WriteScopeMarkerName = "twill_write_scope"`
- `plan:internal/twilldirective#directiveStencil` -> `const directiveStencil = "twill-directive"`
- `plan:internal/twilldirective#writeScopeStencil` -> `const writeScopeStencil = "twill-write-scope"`
- `plan:internal/twilldirective#Lens` -> `type Lens struct`
- `plan:internal/twilldirective#Values` -> `type Values struct`
- `plan:internal/twilldirective#Directive` -> `func Directive(stencilsDir string, v Values) (string, error)`
- `plan:internal/twilldirective#WriteScope` -> `func WriteScope(stencilsDir, readyMarker string) (string, error)`
- `internal/twilldirective/twilldirective_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:contracts/stencils#TwillDirective`
- `plan:contracts/stencils#TwillWriteScope`

**Intent:**
Add `internal/twilldirective`, the small package of decision record D3 that renders the two twill stencils of card 4 from a told stencils directory at call time, the way `internal/editdirective` and `internal/parentdirective` render theirs (`PATTERN-stencil-ownership`).
It imports only the standard library, `internal/stencil` and `internal/stencilstore`, and derives no path; `doc.go` states that import set, what the two blocks say, who renders them and that the package is host-neutral so a later Plan-Write host can render them with its own values.

`Lens` carries `Name` and `Text`, the twill package's own pair so it never imports `burlerengine`; the host converts.
`Values` carries the host-specific marker values: `ReviewedFiles []string`, `ReadyMarker string`, `RubricPath string`, `Lenses []Lens`, `FormatStep string`, `SelfCheckStep string`, `SelfCheckCommand string`, `DispositionPlace string`.

`Directive` reads `directiveStencil` through `stencilstore.Read`, renders `ReviewedFiles` as one backticked bullet per path, `Lenses` as one `- <name>: <text>` bullet per lens with the text's inner newlines collapsed to spaces and trimmed, fills every marker through `stencil.Fill` (none optional), and returns the text without its banner and trailing newlines.
It adds no checks of its own on `Values`: an empty value, or an empty `ReviewedFiles` or `Lenses` list rendered to an empty string, is refused by `stencil.Fill`'s unfilled-marker error, which names the marker.
`WriteScope` reads `writeScopeStencil`, fills `ready_marker` and returns the text the same way.
Each error, the stencil read's and `stencil.Fill`'s, is wrapped naming the stencil.

`twilldirective_test.go` is untagged and seeds a stencils directory with `stencilkit.Seed`: one test over `Directive` pinning that every value lands verbatim (each reviewed file, the marker, the rubric path, each lens name and text, the two steps, the command and the disposition place), that no `{{` remains, and that the text has no leading banner and no trailing newline; one test over `WriteScope` pinning the marker lands and no `{{` remains; and one test pinning that a stencil removed with `stencilkit.Remove` yields an error naming it, for both renderers.

`test-budget.yaml` gains the row `internal/twilldirective` at the test count this card adds, since a package with tests and no row fails the budget test.

**ImpactSummary:** A new leaf package with no callers until card 6; the budget file gains one row.
