# Card 5 — seatengine-prompts

**Create:**
- `internal/seatengine#noneValue`
- `internal/seatengine#strandNames`
- `internal/seatengine#seatValues`
- `internal/seatengine#composePrompt`
- `internal/seatengine#seatSpec`
- `internal/seatengine/prompt_test.go`

**Edit:**
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/agentname#Format`
- `internal/parentdirective#Directive`
- `internal/editdirective#Directive`
- `internal/stencil#FillWith`
- `internal/seatengine#ReservedMarkers`
- `internal/testkit/stencilkit#Seed`

**Intent:** Compose each seat's prompt and shuttle spec (D3, D4, D5, D6, D10) in `prompt.go`.
`strandNames` forms every seat's expected full strand name through `agentname.Format` from the geometry's shortname and slug and the seat's role, the deterministic names seats address each other by (D8).
`seatValues` builds one value map per seat: the table values, then the seat values, then every reserved marker card 4's `ReservedMarkers` lists, so the collision check and the fill read one declaration: `parent_directive` rendered by `parentdirective.Directive` with the run's parent name and the table's interactive flag for the chair, and with the chair's strand name and interactive false for an advisor, which lets a chair message through to the advisor as an operator-grade instruction within the advisor's role, including advice the chair relays from another advisor, bounded never by the chair's word but by the advisor block's own-output write rule (prompt-enforced), the advisor's agent settings' denies and `Interactive` false, so the most it costs is a wrong or wasted advisor output the chair weighs as advice in turn (D6); `edit_directive` from `editdirective.Directive`; `seat_name` and `chair_name` as full strand names; `advisor_names`, `failed_advisors` and `inputs` one per line and the literal `(none)` when empty, with `failed_advisors` always `(none)` for an advisor; `outputs` and `output_files` both the seat's outputs one per line; and the three geometry roots.
`composePrompt` reads the seat's stencil and every block `stencil.IncludeNames` names from the told stencils directory through `stencilstore.Read`, at call time, and fills them with `stencil.FillWith`, so a seat stencil including a seat block renders the block's markers from the seat's value map.
`seatSpec` builds the `shuttleengine.Spec`: the prompt, the seat's outputs, its model triple, `Role: SeatRole(table.RolePrefix, seat.Name)`, the table's segment, the seat's skills and the table's timeout for every seat; `Interactive` from the table for the chair only; `KeepPane` and `QuietHold` set for an advisor only; the chair's gate rides the start call, not the spec.
`prompt_test.go` covers the spec-composition acceptance list against a `stencilkit.Seed` directory, which seeds card 3's registered seat blocks, with a chair stencil including `seat-directive-chair` and an advisor stencil including `seat-directive-advisor`: every reserved marker filled in every seat, the `(none)` cases, the two parent directives naming the right parent, the chair-only interactive flag, the advisor-only `KeepPane` and `QuietHold`, the role, segment, skills and timeout on each spec, and a seat stencil declaring a marker absent from its value map refused by the fill with that marker named.
Raise the `internal/seatengine` budget row by the tests this card adds.

**ImpactSummary:** Only the test budget row changes outside the new unexported helpers; no earlier seatengine symbol or other package changes.
