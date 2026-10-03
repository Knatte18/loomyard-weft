# Card 17 — plankit

**Create:**
- `internal/testkit/plankit/plankit.go`
- `internal/testkit/plankit/plankit_test.go`

**Edit:**
- `CONSTRAINTS.md`
- `docs/overview.md`

**Uses:**
- `plan:internal/planparser#RecognizedFormat`

**Intent:** Add the one test-side writer of valid plans (D10), `internal/testkit/plankit`.
It renders a plan directory from an overview (approved flag, framing, optional `first_card:`, `root:`, `language:`, plan-level sections such as `## verify:`) and cards (number, slug, typed target groups, `Uses:`, `Intent:`, `ImpactSummary:`, optional `Commit:`), writing the format version from `planparser.RecognizedFormat` and the Card Index and card-file names in `planparser`'s grammar.
It exposes the rendered bytes as well as the write, so a rejection test can corrupt one file of an otherwise valid plan without spelling the format itself.
It also writes the file-tree fixture glyph resolution needs (repository-relative path to content under a directory), the helper copied today as `writeFixtureRepo`/`writeGlyphRepo*`.
Its only assertions are `t.Fatalf` on its own setup.
The package doc states the kit's purpose: the one test-side writer of valid plans, rendering through `planparser`'s format constant (D23).
`plankit_test.go` proves that rendered plans parse through `planparser.ParsePlan` and pass `planparser.ValidateFormat` with zero findings across the shapes the kit offers (single and multi-card, approved and not, a `first_card:` offset, multi-label cards with `Uses:`), and that the approved flag round-trips.

`CONSTRAINTS.md`'s Planparser Sole-Parser Invariant is restated per D10: `planparser` is the sole production parser and writer;
in tests, `plankit` is the one writer of valid plans, and a test may write raw plan bytes only to test rejection of a malformed plan.
What this admits, bounded per D10: a second, test-only writer of the plan format, reachable only from `_test.go` files by the Testkit importer rule, rendering through `planparser`'s exported format constant so a format bump fails at one site, with `TestValidate_GoldenFixture_ZeroFindings` staying the parser-side anchor.
`docs/overview.md`'s Tests passage names `plankit`.

**ImpactSummary:** One invariant restated and one overview sentence added alongside the new kit; no existing caller changes in this card.
