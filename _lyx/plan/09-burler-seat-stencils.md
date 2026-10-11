# Card 9 — burler-seat-stencils

**Edit:**
- `contracts/stencils/burler/burler-template-fix-orchestrator.md`
- `contracts/stencils/burler/burler-template-review-orchestrator.md`
- `contracts/stencils/seat/seat-directive-chair.md`
- `contracts/stencils/seat/seat-directive-advisor.md`
- `contracts/stencils#BurlerTemplateReviewOrchestrator`
- `contracts/stencils#BurlerTemplateFixOrchestrator`
- `contracts/stencils/claims_test.go`
- `internal/burlerengine/template_test.go`
- `internal/burlerengine/prompt_test.go`
- `internal/burlerengine/doc.go`

**Intent:**
Turn the two Burler orchestrators into the chair and advisor seat stencils with Burler's own channel rules (decisions D7 and D8), and move the fixer's wait onto `lyx shuttle await`.
The stencil files keep their names and registry entries; only their bodies and header comments change.

Fixer (chair) stencil, `burler-template-fix-orchestrator.md`:
- Step 2 runs `lyx shuttle await {{.ready_marker_path}}` with the Bash tool's `timeout` set to 300000 and repeats it until it reports `ready: true`, editing nothing before then; the sequencing rule names the same verb, and nothing names `lyx burler await-review`.
- The fixer waits inside its turn and never ends its turn to await the reviewer's up line; an up line that lands opens nothing.
- It may message the reviewer, by `SendMessage` to the strand named by `{{.advisor_names}}`, only once the ready marker exists (an existing marker stands in for the up line), and only to ask what an accepted finding means or rests on.
- A reply is advice: the fixer still validates every finding against the code and the fasit, a dispute still needs cited evidence in the fixer-report, and it neither acts on nor fixes anything a reply raises beyond the accepted review.

Reviewer (advisor) stencil, `burler-template-review-orchestrator.md`:
- It names the chair through `{{.chair_name}}`.
- After its review it sends the chair nothing unprompted, never asks the chair to wrap up, answers only by `SendMessage`, writes no file at all, and a reply only explains a finding the accepted review already holds.

Neither stencil includes `seat-directive-chair` or `seat-directive-advisor`, since those blocks' lines contradict these rules (the advisor block lets the chair ask for a wrap-up and the advisor message unprompted; the chair block has the chair end its turn for an up line and read an advisor output as soon as it exists).
Each renders `{{.parent_directive}}` and `{{.edit_directive}}` exactly once, both filled by the seats engine.
The two shared directives' header comments name the Burler seat stencils as the channel wording's other home, replacing their "lives in this block and in … only" sentence.
Each Burler stencil's header comment says it is filled by `internal/seatengine` from the seat's value map, lists its markers (the reserved ones it uses and the seat values `roundTable` gives it), and drops the `composePrompt` and `FillOptional` sentences.
`stencils.go`'s doc comments on the two embedded vars call them the chair and reviewer seat stencils.
These stencil rules state Burler's own, narrower channel and never paraphrase the shared directives (`PATTERN-producer-pointer-rule`).

Bound: the chair sending nothing before the marker, the reviewer sending nothing unprompted and writing no file, and the fixer acting on nothing beyond the accepted review are stencil-enforced only; Go catches a fixer done before the marker and any edit to the review file (card 8).

Tests:
- `contracts/stencils/claims_test.go`: the fix orchestrator's claims swap `lyx burler await-review {{.ready_marker_path}}` for `lyx shuttle await {{.ready_marker_path}}`, add one claim per fixer channel rule above and a `mustNot` on `await-review` and on `{{template "seat-directive-chair"}}`; the review orchestrator's claims add one per reviewer channel rule and a `mustNot` on `{{template "seat-directive-advisor"}}`.
- `internal/burlerengine/template_test.go`: `TestTemplate_FillsWithAllMarkers`'s orchestrator rows gain the seat markers each stencil now names (`chair_name`, `advisor_names`) in their values and required-marker lists.
- `internal/burlerengine/prompt_test.go`: the chair's seat-prompt assertion of the wait line reads `lyx shuttle await` with the ready marker path.

`internal/burlerengine/doc.go`'s fixer paragraph says the fixer waits with `lyx shuttle await` inside its turn, and states the seat channel's rules and their stencil-only bound.

**ImpactSummary:** Two Burler stencil bodies, two shared-directive header comments and their pinned claims and marker tests; no Go behavior changes.
