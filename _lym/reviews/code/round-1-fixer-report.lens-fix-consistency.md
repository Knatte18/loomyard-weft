Lens fix-consistency.

**LOW** — `internal/gatecli/doc.go`, the red-first overlay paragraph.
- The doc states the embed-matching contract but omits the dot and underscore rule for directory patterns and the `all:` prefix.
- It also says nothing on embeds from `testdata` or a dot- or underscore-prefixed directory, which the deleted line contradicted.
- Evidence: `embedPatternMatches` in `internal/gatecli/redfirst.go` and the new testdata row.

**LOW** — `internal/loomshed/redfirstgate.go:33` ("fails with the way forward of a check that cannot judge the diff").
- Generic and still true; the one remaining description of `RedFirstUnjudgedExit` that does not name the non-go scoper. Mention only, not a contradiction.

Checked, no finding: the `isModulePackage` refactor, the `redFirstCmd` Long help, the file header, `baseOverlay`'s doc, `goEmbedDirective` callers, and the renamed row.

Tool calls: 1 (sed of `doc.go` lines 860-900 plus a grep for stale phrases).
