# Card 2 — stencil-include-form

**Edit:**
- `internal/stencil#FillOptional`
- `internal/stencil/stencil_test.go`
- `internal/stencil/export_test.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/stencil#FillWith`
- `internal/stencil#IncludeNames`
- `internal/stencil#includeNames`

**Intent:** Add the stencil include form (D3) beside `Fill` and `FillOptional`, keeping `stencil` a leaf over the standard library.
`FillWith` renders a template together with named included templates: it strips the leading comment of the main template and of every include, parses the main template, and parses each include into the same template set under its name through the parsed template's own `New(name).Parse`, so a `{{template "name"}}` action in the main template resolves to it.
The include form is the bare `{{template "name"}}` D3 writes, which `text/template` would execute with nil data, failing every block marker under `missingkey=error`; so `FillWith` sets each bare include action's pipeline to the root variable `$` in the parsed main tree before executing, and the block renders from the value map `Execute` was given, even inside a `{{with}}` or `{{range}}` body where dot is something else.
Before executing it enforces four rules, each a loud error naming the block: an include action carrying a pipeline of its own, such as `{{template "name" .}}`, is refused, so a seat stencil has one include form; every name `IncludeNames` reports for the main template has an entry in `includes`; no included block itself declares an include, so includes resolve one level deep; and the unfilled-marker guarantee holds over the main template's top-level markers and every include's top-level markers, through the existing `unfilledTopLevelMarkers` walk run over each associated template, with `presentButEmptyBranchMarkers` left to the main template as today.
`FillOptional` becomes a call to `FillWith` with nil includes, so there is one execution path and every registered stencil renders byte-for-byte as before, which the existing marker and claims guards in `contracts/stencils` pin.
`IncludeNames` strips the template's leading comment, as `TopLevelMarkers` does, so a header that spells an include action reports none, then parses it and returns the sorted, deduplicated names of its `{{template "name"}}` actions, walking `{{if}}`, `{{with}}` and `{{range}}` bodies too, so the seat engine knows which blocks to read; `includeNames` is the tree walk it shares with `FillWith`.
`TopLevelMarkers` and every existing caller are unchanged.
`TestFill` gains rows rendered through `FillWith`: a main template whose include renders, a bare include with its own marker filled from the values, a bare include inside a `{{with}}` body rendering its marker from the value map, an include action carrying its own pipeline refused with the block named, an unfilled marker inside an include reported in the error, a missing include named in the error, and a nested include refused;
`export_test.go` gains one test for `IncludeNames` covering a template with no includes, a template whose leading comment spells an include action and whose body has none reporting no names, two includes in different branches, and a parse error.
Raise the `internal/stencil` budget row by the one test this card adds.

**ImpactSummary:** FillOptional's body moves behind FillWith with identical output for a nil include map; the two new exported functions have no caller until card 5.
