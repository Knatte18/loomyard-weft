# Card 5 — drift-expectation-parser

**Create:**
- `plan:internal/driftengine#Expectation` -> `type Expectation struct`
- `plan:internal/driftengine#ExpectedCase` -> `type ExpectedCase struct`
- `plan:internal/driftengine#KeyEntry` -> `type KeyEntry struct`
- `plan:internal/driftengine#LineEntry` -> `type LineEntry struct`
- `plan:internal/driftengine#CaseEntry` -> `type CaseEntry struct`
- `plan:internal/driftengine#Coverage` -> `type Coverage struct`
- `plan:internal/driftengine#ParseExpectation` -> `func ParseExpectation(data []byte, mode string, baseline []CaseListing) (Expectation, error)`
- `plan:internal/driftengine#Expectation.Merge` -> `func (e Expectation) Merge(more Expectation) Expectation`
- `plan:internal/driftengine#Expectation.Cover` -> `func (e Expectation) Cover(c Cell) (Coverage, bool)`
- `plan:internal/driftengine#Expectation.Uncovering` -> `func (e Expectation) Uncovering(cells []Cell) []string`
- `plan:internal/driftengine#TestParseExpectation` -> `func TestParseExpectation(t *testing.T)`
- `plan:internal/driftengine#TestExpectationCover` -> `func TestExpectationCover(t *testing.T)`

**Extends:**
- `internal/driftengine/doc.go`

**Uses:**
- `plan:internal/driftengine#CaseListing`
- `plan:internal/driftengine#Cell`
- `plan:internal/driftengine#DiffLines`
- `plan:internal/driftengine#DiffCommand`

**Intent:** The sole parser of `_lym/drift/expected.yaml` (`PATTERN-sole-parsers`, D5, D2): an agent-written contract file, decoded strictly, so an unknown key is refused (`PATTERN-tolerant-records` keeps tolerance for Go-written records only).

The file's shape: `cases`, a map from case name to `ExpectedCase`; `added` and `removed`, lists of `CaseEntry` (`case`, `why`).
Under `lines`, an `ExpectedCase` holds `keys`, a list of `KeyEntry` (`key`, optional `n`, optional `added: true`, `why`), and `lines`, a list of `LineEntry` (`range`, `why`) whose range is `12`, `12-20` or the open `12-`.
Under `command` an `ExpectedCase` holds one `why` and nothing else.
An empty file and an empty document parse as the empty expectation, which is valid.

`ParseExpectation` refuses, each naming the entry and the rule, with no way-forward clause since the expect agent's `expectation` entry re-prompts on it:
- under `lines`: a case neither in `baseline` nor under `added`; a `removed` case absent from `baseline`; a `keys` entry whose key, or key and `n`, the case's baseline listing lacks, unless it carries `added: true`; a range that is not `N`, `N-M` with `1 <= N <= M`, or `N-`; a case-level `why`;
- under `command`: a `keys` or `lines` entry, and no case name is checked against `baseline`;
- under both: an empty or blank `why`, an `n` below 1, and an unknown key.

`Merge` appends `more`'s entries to the receiver's, the way containment reads the sealed file and every amendment as one expectation (D11).
`Cover` returns the first entry covering a cell, its `why` and whether it is an open range: an `added case` cell by an `added` entry, a `removed case` cell by a `removed` entry, a keyed cell by a `keys` entry of its case with its key and either no `n` or its `N`, an unkeyed cell by a `lines` entry whose range holds its `Line`; a `lines` entry never covers a keyed cell.
`Uncovering` names every entry that covers none of `cells`, for the verdict (D7, D13).

`TestParseExpectation` (tier0) is table-driven over D5's and D2's criteria: a key with and without `n`, closed and open ranges, `added` and `removed` parse; an unknown case, a `removed` case missing from the baseline, an unknown key not marked `added`, a malformed range, an empty `why` and an unknown field are refused; under `command` a `keys` and a `lines` entry are refused and a case absent from the baseline parses.
`TestExpectationCover` (tier0) covers: a key entry without `n` covers every occurrence and one with `n` covers that one only; a line entry covers an unkeyed cell in its range, an open range every later one, and no keyed cell; added and removed case cells; `Merge` makes an amendment's entry cover a cell the sealed file did not; `Uncovering` lists an entry with no cell.
The doc.go gains an `# Expectation` section naming the parser as sole reader.

**ImpactSummary:** New parser and model in driftengine; nothing existing calls or changes.
