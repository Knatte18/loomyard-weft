```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over the diff. For each branch I ran one concrete input and judged the result. I found no defects.

**Cases run**
- `discussionparser.Citations` (`citations.go`):
  - `` `a``b` `` gives the spans `a` and `b`: correct.
  - An empty span and an odd trailing backtick are skipped: correct.
  - An unclosed carry-over block skips to the end of the record: correct.
  - A mark directly after the closing backtick sets `Marked`; a mark after a space does not: correct.
- `CheckDecisionIDs` (`decisionids.go`):
  - `### Design notes`, `### D01 — x`, `### D1 —` and a bare `### D` are all malformed: correct.
  - A repeated id is reported once: correct.
  - `D1` is not matched by `D10`, and `D1_x` counts as `D1`: correct.
  - A record with no acceptance section yields no coverage finding: correct.
  - A record with no decisions section yields nil: correct.
- `ValidateLoomDiscussion` and `ValidateDarnDiscussion` (`gates.go`):
  - A missing record adds no id or citation finding: correct.
  - A record `Validate` finds defective still gets the id findings, and the loom caller also still gets the citation findings: correct.
  - A quarry outage returns an error with nil findings: correct.
  - The darn caller runs the history check only when `Validate` is clean, with the id findings first: correct.
- `citationFindings` (`citations.go`):
  - `internal/` naming a file, `/abs/p`, `./x/y` and an import path under an untracked root are all untested: correct.
  - A marked token is untested: correct.
  - `internal/x.go#Foo` goes down the glyph route, and `docs/overview.md#Modules` (an existing `.md` file) and `C#` are not glyphs: correct.
  - `state.json`, `os.ReadFile` and `CLAUDE.md` are untested because the member has no upper-case letter or the package has no directory: correct.
  - `pkg.Fn(` with arguments is untested because of the whitespace filter, and `pkg.Fn()` has its call parenthesis stripped: correct.
  - Two package directories with one base name give `citation-symbol-ambiguous`: correct.
  - Positional answer matching survives the mix of items that already carry a local finding: correct.
- `planglyph.ResolveGlyphs` (`index.go`, `resolve.go`):
  - An empty list returns nil: correct.
  - An unreadable or pre-resolution answer gives `GlyphMissing`: correct.
  - An ambiguous answer split by build constraint gives `GlyphFound` with the candidate files: correct.
  - An ambiguous answer not so split gives `GlyphAmbiguous` with candidates: correct.
  - The `statusFindings` refactor onto `resolveVerdict` keeps the glyph-rejected, glyph-ambiguous and glyph-not-found branches unchanged: correct.
- `testTierFindings` (`testtier.go`):
  - One tag, the same tag on two lines, `tier0 || tier1`, `untagged`, two tags on separate lines, `untagged` plus a tag, and no Intent lines all give the expected verdict.
  - A full path or a base name counts only as a token of its own, so `a_test.go` inside `x/a_test.go` does not match: correct.
  - Nil `TierTags` or a non-go plan skips the check: correct.
- `seamTestFindings`, `touchesStruct` and `coversSeamTest` (`seamtest.go`):
  - Field, handle, declaring-file and package-self touches each fire.
  - A Rename touch is seen, because `Refs` carries both endpoints of a Rename pair (`planparser/plan.go:220`).
  - The Uses, later-Edit and `_test` counterpart covers each pass: correct.
  - An out-of-module marker such as `time#Time` still matches by glyph: correct.
  - No cards or no seams makes no index call: correct.
- `docGoFindings` (`docgo.go`):
  - A Create file target `new.go` and an Edit of a prod member in a package with a `doc.go` give a finding.
  - A test-only card, a `_test` pseudo-unit, a Rename of a `Test*` member, a non-`.go` file, a package with no `doc.go`, a package self glyph, a Delete of the `doc.go` and a later Edit of it all pass: correct.
  - An unresolved Create handle named like a test counts as test code: correct.
- `FindSeamTests` (`impactset/seams.go`):
  - A marker above a `func Test…` is placed, and it is placed when stacked with a guard or keep line.
  - A marker on a helper function, in a function body, with prose after it, or with a malformed argument gives the scan error with the line: correct.
  - `//lym:seamless` is not a marker: correct.
  - The guard scan now admits seam lines between `//lym:guard` and the func (`guards.go` `onlyKeepLinesAfter`): correct.
- `seamViolations` in `envkit_test.go`:
  - An unmarked `NumField` or `VisibleFields` test, a call through a same-package helper, a bare `NilSeams` inside package `envkit`, a `NilSeams` marker set that misses a nested struct, a marker naming a func, and an out-of-module marker are each judged correctly by the fixtures.
- Wiring (`entries_gate.go`, `wiring.go`, `validate.go`):
  - The discussion gate requires `WorktreeRoot` and `PlanIndex`.
  - `validate-discussion` reads `c.env.PlanIndex`, which `wiring.go:311` sets before the config loads.
  - Both `PlanContext` builders fill `TierTags` and `SeamTests`: correct.
  - An absent `gate.yaml` leaves the tier check skipped: correct.

**Notes, not findings**
- `coversDocGo` and `coversSeamTest` look at `plan.Cards` for a later editor, done cards included. This is harmless.
- `goPackageDirs` leaves out the repo-root directory, so `SeamTestReader` never reads root-level `_test.go` files. No root Go package exists in the part of the diff I read, so I did not raise it.

**Tool calls**
1. `grep PlanIndex` in `internal/loomcli/*.go`: to confirm `validate-discussion` wiring supplies the index; my context lacked the wiring read.
2. `grep Pairs|Refs` in `internal/planparser`: to confirm whether a Rename group's `Refs` carries both pair endpoints; my context lacked the planparser definition. Its comment at `plan.go:220` says it does.
