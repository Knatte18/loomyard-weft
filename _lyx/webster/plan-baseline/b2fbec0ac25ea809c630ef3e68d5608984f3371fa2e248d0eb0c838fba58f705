# Card 3 — planparser-symbol-renames

**Create:**
- `plan:internal/planparser#SymbolRenamePairs` -> `func SymbolRenamePairs(plan *Plan, cards []Card) []MovePair`
- `plan:internal/planparser#gateUntypedFileRenamePair` -> `const gateUntypedFileRenamePair refGate = "untyped-file-rename-pair"`
- `plan:internal/planparser_test#TestSymbolRenamePairs` -> `func TestSymbolRenamePairs(t *testing.T)`

**Edit:**
- `internal/planparser#ledger`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/planparser#isFileRenamePair`
- `internal/planparser#planLanguage`
- `internal/planparser#lookup`

**Intent:**
The one classifier D1.7's plan-gate check and D1.8's `begin-batch` and `recover-batch` refusal share, kept in `internal/planparser` because `PATTERN-ref-shape-registry` makes it the sole place a plan ref's shape is decided.

`SymbolRenamePairs` returns, in card order and pair order, every pair of every `cards` entry's `Pairs` that is a symbol pair: every pair but a file pair.
Under a language `planLanguage` accepts, a pair is a file pair when `isFileRenamePair` reports it, both sides file self glyphs.
Under a language it rejects, such as `none`, a pair is a file pair when both sides classify as file paths, decided through `lookup` with the new `gateUntypedFileRenamePair` row, which keeps `refKindPath` and skips every other kind; that row joins `ledger`.
It returns nil when no card holds a symbol pair, and never reads the disk.
`plan` supplies only the language; callers pass the cards they check, the whole plan's for the gate and a batch's for webster.

`TestSymbolRenamePairs` is table-driven over a `Plan` literal and its cards: a member-glyph old side with a `plan:` handle new side is returned; a file-self-glyph pair is not; a card with both returns only the symbol pair; a plan with no `Rename` card returns nil; under `language: none` a two-path pair is not returned and a pair of bare names is; cards from two cards keep card order.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/planparser` row rises by one for it.

**ImpactSummary:** one new exported reader and one new ledger row; no existing check changes behavior.
