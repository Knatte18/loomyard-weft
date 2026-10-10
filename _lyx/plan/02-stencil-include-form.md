# Card 2 — stencil-include-form

**Edit:**
- `internal/stencil#FillOptional`
- `internal/stencil/stencil_test.go`
- `internal/stencil/export_test.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/stencil#FillWith` -> `func FillWith(template []byte, includes map[string][]byte, values map[string]string, optional []string) ([]byte, error)`
- `plan:internal/stencil#IncludeNames` -> `func IncludeNames(template []byte) ([]string, error)`
- `plan:internal/stencil#includeNames` -> `func includeNames(root *parse.ListNode) []string`

**Intent:** Add the stencil include form (D3) beside `Fill` and `FillOptional`, keeping `stencil` a leaf over the standard library.
`FillWith` renders a template together with named included templates: it strips the leading comment of the main template and of every include, parses the main template, and parses each include into the same template set under its name through the parsed template's own `New(name).Parse`, so a `{{template "name"}}` action in the main template resolves to it.
Before executing it enforces three rules, each a loud error naming the block: every name `IncludeNames` reports for the main template has an entry in `includes`; no included block itself declares an include, so includes resolve one level deep; and the unfilled-marker guarantee holds over the main template's top-level markers and every include's top-level markers, through the existing `unfilledTopLevelMarkers` walk run over each associated template, with `presentButEmptyBranchMarkers` left to the main template as today.
`FillOptional` becomes a call to `FillWith` with nil includes, so there is one execution path and every registered stencil renders byte-for-byte as before, which the existing marker and claims guards in `contracts/stencils` pin.
`IncludeNames` parses a template and returns the sorted, deduplicated names of its `{{template "name"}}` actions, walking `{{if}}`, `{{with}}` and `{{range}}` bodies too, so the seat engine knows which blocks to read; `includeNames` is the tree walk it shares with `FillWith`.
`TopLevelMarkers` and every existing caller are unchanged.
`TestFill` gains rows rendered through `FillWith`: a main template whose include renders, an include with its own marker filled from the values, an unfilled marker inside an include reported in the error, a missing include named in the error, and a nested include refused;
`export_test.go` gains one test for `IncludeNames` covering a template with no includes, two includes in different branches, and a parse error.
Raise the `internal/stencil` budget row by the one test this card adds.

**ImpactSummary:** FillOptional's body moves behind FillWith with identical output for a nil include map; the two new exported functions have no caller until card 5.
