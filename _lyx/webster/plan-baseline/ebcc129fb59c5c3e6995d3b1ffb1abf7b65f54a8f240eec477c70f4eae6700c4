# Card 1 — seat-table-optional-values

**Edit:**
- `internal/seatengine#Table`
- `internal/seatengine#Table.Validate`
- `internal/seatengine#composePrompt`
- `internal/seatengine#Engine.startSeat`
- `internal/seatengine/table_test.go`
- `internal/seatengine/prompt_test.go`
- `internal/seatengine/doc.go`

**Create:**
- `internal/seatengine#composeSeatPrompt`
- `internal/seatengine#Prompts`

**Uses:**
- `internal/seatengine#seatValues`
- `internal/seatengine#validateValues`
- `internal/stencil#FillWith`

**Intent:** Let a seat table name the table values that may be empty (D11), and expose the composed prompts so a table builder can test what its seats will read.
Every shipped signature in this package stays as it is, since `burler-multi-llm` edits it in parallel and the Constraints allow only additive changes here: no function is re-signed, and the new behaviour arrives through a new field and two new functions.
`Table` gains an `Optional []string` field: the table value names that may be blank.
`Table.Validate` refuses a listed name that is also a reserved marker, naming it, then hands `validateValues` a copy of the table's values with every listed name whose value is blank dropped, so a listed name is exempt from the blank-value rule while every unlisted table value keeps it; seat values go to `validateValues` unchanged, so a blank seat value is still refused whether or not its name is listed.
`composeSeatPrompt` carries `composePrompt`'s body with the optional list handed to `stencil.FillWith` as its optional markers, so an empty listed value renders as nothing in the stencil and in every block it includes; `composePrompt` keeps its signature and calls `composeSeatPrompt` with no optional names, and `Engine.startSeat` calls `composeSeatPrompt` with `table.Optional`.
`Prompts` is the pure composition of every seat's prompt, keyed by seat name: it validates the table against `geom.StencilsDir`, forms the strand names, builds each seat's values through `seatValues` with no failed advisors, and composes each prompt through `composeSeatPrompt` with the table's optional list; it starts nothing.
Bound: only a listed table value may be blank; every unlisted table value, every seat value and every reserved marker keeps the blank-value refusal and the fill's missing-key refusal.
Rows in `TestTable_Validate`: a listed table value may be blank, an unlisted blank table value is still refused, a listed name that is a reserved marker is refused naming it, and a blank seat value is still refused whether or not its name is listed.
A subtest in `TestSeatPrompts_ValuesPromptsAndSpecs` pins that `Prompts` returns one prompt per seat, equal to what the existing `composePrompt` assertions compose for that seat, and that a listed empty table value renders as nothing in a stencil and in an included block that carries its marker; the existing `composePrompt` calls are unchanged.
The package doc's table-rules section names the optional list.

**ImpactSummary:** No shipped signature changes: one struct field and two functions are added, `composePrompt` keeps its signature and callers, and every existing table keeps its behaviour with an empty optional list.
