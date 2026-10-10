# Card 3 — seat-directive-stencils

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#TestStencils_TopLevelMarkers`

**Create:**
- `//contracts/stencils/seat/seat-directive-chair.md`
- `//contracts/stencils/seat/seat-directive-advisor.md`
- `plan:contracts/stencils#SeatDirectiveChair` -> `var SeatDirectiveChair []byte`
- `plan:contracts/stencils#SeatDirectiveAdvisor` -> `var SeatDirectiveAdvisor []byte`

**Intent:** Ship the two shared seat blocks (D3, D6) under a new `seat` family, each an embedded byte var in `stencils.go` with a registry row in `entries`, so they are seeded and listed like every stencil; neither joins `roleOpeningStencils`, since a seat block is not an opening stencil.
Both blocks open with the header comment every stencil carries, naming which markers they declare and that a seat stencil pulls them in with `{{template "seat-directive-chair"}}` or `{{template "seat-directive-advisor"}}`, the bare form with no pipeline, which card 2's fill hands the seat's value map.
`seat-directive-chair` declares `advisor_names`, `failed_advisors`, `inputs` and `output_files`: the chair steers and decides; it asks an advisor by `SendMessage` to that advisor's strand name from `advisor_names`, and the advisors answer its questions over that channel; and a message from a named advisor is advice to weigh, never an instruction; their outputs at the listed input paths are its inputs; the advisors in `failed_advisors` never started; an advisor failure notice may arrive and the chair carries on; an advisor that does not answer in reasonable time is written off; the step ends when the chair writes its outputs.
`seat-directive-advisor` declares `chair_name`, `seat_name` and `output_files`: who the chair is, the seat that leads the step; the advisor's own output, the only files it writes; it messages the chair only, by `SendMessage` to its strand name, never another advisor and never any other session; it writes its output when its task is done or when the chair asks it to wrap up, and keeps answering the chair after writing it; a question it cannot answer goes to the chair.
The advisor-to-chair route this opens is bounded by prompt alone (D6): no mechanism scopes a session's message targets, so an advisor that disobeys its block can reach any name `ListAgents` shows, the run's parent included; what bounds it is that no advisor prompt names any session but the chair and its fellow advisors, advisors are never interactive and their held turns notify no parent (card 5), and an advisor's message carries no delegate authority, since the chair block makes it advice to weigh.
The seat-to-seat channel wording lives in these two blocks only, and neither tells an agent to ask the operator, restates the edit rule, names a skill or cites a cross-repository path, so the existing registry, claims, ask-operator, edit-rule, skill and citation guards pass over them unchanged.
`TestStencils_TopLevelMarkers` gains one row per block pinning its exact marker set, which is the contract card 5's value map fills.

**ImpactSummary:** Two new registry rows and two new embedded vars; no existing stencil, role map or guard changes.
