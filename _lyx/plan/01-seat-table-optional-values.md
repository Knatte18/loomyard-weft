# Card 1 — seat-table-optional-values

**Edit:**
- `internal/seatengine#Table`
- `internal/seatengine#Table.Validate`
- `internal/seatengine#validateValues` -> `func validateValues(owner string, values map[string]string, optional []string) error`
- `internal/seatengine#composePrompt` -> `func composePrompt(stencilsDir string, seat Seat, values map[string]string, optional []string) (string, error)`
- `internal/seatengine#Engine.startSeat`
- `internal/seatengine/table_test.go`
- `internal/seatengine/prompt_test.go`
- `internal/seatengine/doc.go`

**Create:**
- `plan:internal/seatengine#Prompts` -> `func Prompts(geom Geometry, table Table) (map[string]string, error)`

**Uses:**
- `internal/seatengine#seatValues`

**Intent:** Let a seat table name the table values that may be empty (D11), and expose the composed prompts so a table builder can test what its seats will read.
`Table` gains an `Optional []string` field: the table value names that may be blank.
`Table.Validate` passes it to `validateValues` for the table's values, so a listed name is exempt from the blank-value rule, and refuses a listed name that is also a reserved marker, naming it; seat values keep the blank-value refusal unchanged, so `Validate` passes `nil` for them.
`composePrompt` takes the optional list and hands it to `stencil.FillWith` as its optional markers, so an empty listed value renders as nothing; `Engine.startSeat` passes `table.Optional`.
`Prompts` is the pure composition of every seat's prompt, keyed by seat name: it validates the table against `geom.StencilsDir`, forms the strand names, builds each seat's values through `seatValues` with no failed advisors, and composes each prompt with the table's optional list; it starts nothing.
It is additive, since `burler-multi-llm` edits this package in parallel.
Rows in `TestTable_Validate`: a listed table value may be blank, an unlisted blank table value is still refused, a listed name that is a reserved marker is refused naming it, and a blank seat value is still refused whether or not its name is listed.
A subtest in `TestSeatPrompts_ValuesPromptsAndSpecs` pins that a listed empty value renders as nothing through `composePrompt` and that `Prompts` returns one prompt per seat, equal to what `composePrompt` composes for that seat; the existing `composePrompt` calls pass `nil`.
The package doc's table-rules section names the optional list.

**ImpactSummary:** Two unexported re-signs with callers inside this package only; the new field and function are additive, and every existing table keeps its behaviour with an empty optional list.
