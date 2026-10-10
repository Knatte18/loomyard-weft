# Card 4 — seatengine-table

**Create:**
- `internal/seatengine/doc.go`
- `internal/seatengine#RoleChair`
- `internal/seatengine#RoleAdvisor`
- `internal/seatengine#AdvisorName`
- `internal/seatengine#SeatRole`
- `internal/seatengine#Seat`
- `internal/seatengine#Table`
- `internal/seatengine#ReservedMarkers`
- `internal/seatengine#Table.Validate`
- `internal/seatengine#Table.Chair`
- `internal/seatengine#Table.Outputs`
- `internal/seatengine#Geometry`
- `internal/seatengine#Handle`
- `internal/seatengine#Shuttle`
- `internal/seatengine#runnerShuttle`
- `internal/seatengine#RunnerShuttle`
- `internal/seatengine/table_test.go`

**Edit:**
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/burlerengine#Handle`
- `internal/burlerengine#Shuttle`
- `internal/burlerengine#RunnerShuttle`
- `internal/agentname#ValidateRole`
- `internal/stencil#IncludeNames`
- `internal/stencilstore#Read`

**Intent:** Open the new package `internal/seatengine` (D1) with its declarations: the table, its validation, the told geometry and the handle-shaped shuttle seam, in `table.go`, `geometry.go` and `shuttle.go`.
`Seat` carries `Name`, `Stencil`, `Model`, `Effort`, `Version`, `Inputs`, `Outputs`, `Skills` and `Values`; `Table` carries `RolePrefix`, `Segment` (a `segmentcolor.Segment`), `Gate` (a `shuttleengine.GateSpec`, the chair's), `Timeout`, `Interactive`, `Values` and the ordered `Seats` (D2).
`AdvisorName` forms `advisor-<n>`, `SeatRole` forms `<prefix>-<seat name>` (D4), and the two constants are the role names this module owns, so no spec literal ever spells a role.
`Table.Validate` applies every D2 rule, each a loud error naming the seat: exactly one seat named `chair`; every other seat named `advisor-<n>` with `n` running from 1 contiguously in table order; every seat has at least one output and no two seats share an output path; every advisor output is among the chair's inputs; the role prefix and every seat's formed role pass `agentname.ValidateRole`; every seat's stencil is readable from the told stencils directory and every block it includes (through `stencil.IncludeNames` over the read bytes) is readable too and declares no include of its own; and a `Values` name, on the table or a seat, that collides with a reserved marker or carries an empty or whitespace-only value is refused.
`ReservedMarkers` returns D3's reserved marker names in one sorted list, `parent_directive`, `edit_directive`, `seat_name`, `chair_name`, `advisor_names`, `failed_advisors`, `inputs`, `outputs`, `output_files`, `worktree_root`, `anchor_path` and `stencils_dir`, from one package-level declaration in `table.go` that `Validate`'s collision check reads and card 5's value map fills.
`Table.Chair` returns the chair seat and `Table.Outputs` every seat's outputs in table order, the two reads the adapter needs.
`Geometry` is the told struct: `WorktreeRoot`, `AnchorPath`, `StencilsDir`, `ParentName`, `Shortname` and `Slug`, all absolute paths or plain names a caller supplies, and the package derives none (PATTERN-told-geometry).
`Handle` is burler's handle plus `Send(text string) error` and `StrandName() string`, which `*shuttleengine.Run` satisfies after card 1, pinned by a compile-time assertion; `Shuttle` is `StartGated` and `ProbeGated` with the same interface-nil discipline burler's `runnerShuttle` documents, and `RunnerShuttle` adapts a `*shuttleengine.Runner`.
The package imports only `shuttleengine`, `stencil`, `stencilstore`, `agentname`, `parentdirective`, `editdirective`, `segmentcolor`, `logger` and the standard library; `doc.go` states the seat model, the table rules and the told geometry, and names the import set, which later cards extend with the run, notice, probe and resume rules.
`table_test.go` is one table-driven test over `Validate` covering every refusal above and the accepted table, against a `stencilkit.Seed` directory holding a chair stencil and an advisor stencil written by the test, plus one test for `AdvisorName`, `SeatRole`, `Chair`, `Outputs` and `ReservedMarkers`.
Add the `internal/seatengine` budget row sized for the tests this card adds.

**ImpactSummary:** A new package with no caller yet; the only existing file touched is the test budget, which gains one row.
