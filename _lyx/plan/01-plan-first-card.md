# Card 1 — plan-first-card

**Edit:**
- `internal/planparser/plan.go`
- `internal/planparser/parse.go`
- `internal/planparser/validate.go`
- `internal/planparser/parse_test.go`
- `internal/planparser/validate_test.go`
- `internal/planparser/approve_test.go`

**Prosa:**
- `contracts/specs/loom-plan-spec.md`

**Intent:**
Give the plan format an optional `first_card` scalar in `00-overview.md` frontmatter (D3), so a rework generation can number its cards on from the generation it retires.

- `Plan` gains `FirstCard int`, `1` when the key is absent, and a record of an invalid value (the raw text) for the check to report.
- `overviewFrontmatter` gains the key, decoded so that a non-integer value does not fail the strict `KnownFields` decode: decode it as a `yaml.Node` (or equivalent) and interpret it in `ParsePlanFrom`, so a non-integer `first_card` reaches validation as a finding rather than a parse error.
- `checkIndexFileConsistency` (`index-file-mismatch`) checks the Card Index numbering against `first_card .. first_card+n-1` instead of `1..n`, so a gap or a card below `first_card` is a finding.
  A non-positive or non-integer `first_card` is an `index-file-mismatch` finding of its own naming the value, and the numbering half then checks against `1`.
  No new check ID is added, so the check-ID set and every count that names it stay unchanged.
- `card-numbering` keeps comparing each card heading against its Card Index number, which `index-file-mismatch` now anchors at `first_card`; the doc comments of both checks say so.
- `first_card` is parsed and checked only here, per the Planparser Sole-Parser Invariant; `SetApproved` rewrites only the `approved:` line, so it preserves `first_card` untouched.

Tests (table-driven, in the existing test files): absent `first_card` behaves as `1`; a plan numbered `7, 8, 9` with `first_card: 7` validates clean; a gap (`first_card: 7` with cards `7, 9`), a card below `first_card` (`first_card: 7` with cards `6, 7`), `first_card: 0`, `first_card: -2` and `first_card: seven` each produce an `index-file-mismatch` finding; `SetApproved` over an overview carrying `first_card: 7` leaves that line in place.

`loom-plan-spec.md` documents the key in the on-disk layout's frontmatter block, in "Numbering and commit subject" (card numbers run from `first_card`, default `1`, with no gap) and in the `index-file-mismatch` row.

**ImpactSummary:** planparser-internal: one new frontmatter key and a numbering base inside one existing check; every plan without the key parses and validates exactly as before.
