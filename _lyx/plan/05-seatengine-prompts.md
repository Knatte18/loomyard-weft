# Card 5 — seatengine-prompts

**Create:**
- `plan:internal/seatengine#ReservedMarkers` -> `func ReservedMarkers() []string`
- `plan:internal/seatengine#noneValue` -> `const noneValue = "(none)"`
- `plan:internal/seatengine#strandNames` -> `func strandNames(geom Geometry, table Table) (map[string]string, error)`
- `plan:internal/seatengine#seatValues` -> `func seatValues(geom Geometry, table Table, seat Seat, names map[string]string, failed []string) (map[string]string, error)`
- `plan:internal/seatengine#composePrompt` -> `func composePrompt(stencilsDir string, seat Seat, values map[string]string) (string, error)`
- `plan:internal/seatengine#seatSpec` -> `func seatSpec(table Table, seat Seat, prompt string) shuttleengine.Spec`
- `internal/seatengine/prompt_test.go`

**Edit:**
- `plan:internal/seatengine#Table.Validate`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/agentname#Format`
- `internal/parentdirective#Directive`
- `internal/editdirective#Directive`
- `plan:internal/stencil#FillWith`
- `plan:contracts/stencils#SeatDirectiveChair`
- `plan:contracts/stencils#SeatDirectiveAdvisor`

**Intent:** Compose each seat's prompt and shuttle spec (D3, D4, D5, D6, D10) in `prompt.go`.
`ReservedMarkers` returns the reserved marker names in one sorted list, `parent_directive`, `edit_directive`, `seat_name`, `chair_name`, `advisor_names`, `failed_advisors`, `inputs`, `outputs`, `output_files`, `worktree_root`, `anchor_path` and `stencils_dir`, backed by the package-level list card 4 declared; `Table.Validate` now reads that list through the same declaration, so the collision check and the fill never drift.
`strandNames` forms every seat's expected full strand name through `agentname.Format` from the geometry's shortname and slug and the seat's role, the deterministic names seats address each other by (D8).
`seatValues` builds one value map per seat: the table values, then the seat values, then every reserved marker: `parent_directive` rendered by `parentdirective.Directive` with the run's parent name and the table's interactive flag for the chair, and with the chair's strand name and interactive false for an advisor; `edit_directive` from `editdirective.Directive`; `seat_name` and `chair_name` as full strand names; `advisor_names`, `failed_advisors` and `inputs` one per line and the literal `(none)` when empty, with `failed_advisors` always `(none)` for an advisor; `outputs` and `output_files` both the seat's outputs one per line; and the three geometry roots.
`composePrompt` reads the seat's stencil and every block `stencil.IncludeNames` names from the told stencils directory through `stencilstore.Read`, at call time, and fills them with `stencil.FillWith`, so a seat stencil including a seat block renders the block's markers from the seat's value map.
`seatSpec` builds the `shuttleengine.Spec`: the prompt, the seat's outputs, its model triple, `Role: SeatRole(table.RolePrefix, seat.Name)`, the table's segment, the seat's skills and the table's timeout for every seat; `Interactive` from the table for the chair only; `KeepPane` and `QuietHold` set for an advisor only; the chair's gate rides the start call, not the spec.
`prompt_test.go` covers the spec-composition acceptance list against a seeded stencils directory with a chair stencil including `seat-directive-chair` and an advisor stencil including `seat-directive-advisor`: every reserved marker filled in every seat, the `(none)` cases, the two parent directives naming the right parent, the chair-only interactive flag, the advisor-only `KeepPane` and `QuietHold`, the role, segment, skills and timeout on each spec, and a seat stencil missing a reserved marker refused by the fill.
Raise the `internal/seatengine` budget row by the tests this card adds.

**ImpactSummary:** Validate's reserved-name check now reads the list ReservedMarkers wraps; no other symbol or package changes.
