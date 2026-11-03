---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/loomcli/darnwiring.go:261
    summary: only the Darn-Write table is checked at Preflight, though the entry asks Preflight to reject a seat whose stencil markers cannot all be filled
    fix: 'Either state in the PreflightStencils doc on internal/shedrecipe/recipe.go that only the Darn-Write writer table is checked because the other seat tables are built from call-time values Preflight does not have, or defer the remaining tables to the run parent to record with lym loom decision add. No code change is needed if the narrowing is accepted.'
  - id: F2
    severity: LOW
    class: consistency
    location: internal/seatengine/prompt.go:145
    summary: the CheckMarkers refusal always says the stencils are newer than this lym binary, which misleads when the unfilled marker is a blank non-Optional value rather than a stencil skew
    fix: 'In Table.CheckMarkers, word the error so it names both causes: the stencils in the directory are newer than this lym binary, or a value is blank. Give a way forward for each: re-step with a lym binary that fills them, or give the blank value text. Update the matching substrings in TestTable_CheckMarkers and in the CheckMarkers and doc.go comments.'
---

### [LOW] Only the Darn-Write table is checked at Preflight

**Location:** internal/loomcli/darnwiring.go:261
**Issue:** The amended entry asks that Preflight, or the recipe loader, reject a seat whose stencil markers cannot all be filled.
The diff wires `PreflightStencils` only in `wireDarn`, so the check covers the darn writer table.
The other seat tables, among them the burler, discussion, plan and drift tables, are still found out inside the seat, now with the seat's role named.
The incident was the Darn-Write seat, and those tables take call-time values Preflight cannot build, so the narrowing is defensible.
It is not stated anywhere, though.

### [LOW] The unfilled-marker refusal blames version skew for a blank value

**Location:** internal/seatengine/prompt.go:145
**Issue:** `CheckMarkers` counts a blank non-Optional table or seat value as unfilled, which matches `FillWith`.
The error text then says the stencils are newer than this lym binary and that the way forward is a newer binary.
That is wrong when the cause is a blank value.
`loom.yaml` refuses an empty `subagent_model` at load, so the path is unlikely today.
The wording would still send an operator the wrong way.

## Focus answers

- `PreflightStencils` closure, unfilled marker: `the wired stencils check refuses a marker of the writer stencil no value fills` appends `{{.unfilled_marker}}` to the seeded `darn-template-write` and asserts `seat "darn"`, the stencil name and the marker.
  A closure returning nil fails it. Not a defect.
- Same closure, `darnTable` error: it wraps with "build the darn writer table" and a fix-and-re-step way forward. The reason prefix `seat stencils check failed: ` is neutral, so a table-build fault is not called unfillable.
  The unresolvable model-spec test covers it. Not a defect.
- `Table.fills` and `CheckMarkers`: the seat value wins over the table value, a blank value is unfilled, and Optional and reserved markers are exempt.
  All of this matches `stencil.unfilledTopLevelMarkers` and the `maps.Copy` order in `seatValues`. Not a defect.
- `preflightEntry`: it forwards `env.PreflightStencils`, and `Preflight forwards the stencils check` fails if the forwarding is dropped. Not a defect.
- Seat tests: the lead's role-named compose error is pinned by exact string. The `CheckMarkers` rows cover a parse error, a marker in both stencil and block, and first-failing-seat ordering. Not a defect.
- `preflight.go` header: it names `NewPreflight` and `NewPreflightWithChecks`, both of which exist. Not a defect.
