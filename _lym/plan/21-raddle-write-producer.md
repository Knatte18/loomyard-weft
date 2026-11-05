# Card 21 — raddle-write-producer

**Create:**
- `contracts/stencils/raddle/raddle-template-agent.md`
- `plan:contracts/stencils#RaddleTemplateAgent` -> `var RaddleTemplateAgent []byte`
- `plan:internal/loomengine#raddleRole` -> `const raddleRole = "raddle"`
- `plan:internal/loomengine#RaddleTable` -> `func RaddleTable(layout *lymcwd.Location, stencilsDir, specsDir string, cfg Config, reg modelspec.Registry, slug string, interactive bool, returnReason string) (seatengine.Table, error)`
- `plan:internal/loomengine#TestRaddleTable` -> `func TestRaddleTable(t *testing.T)`
- `plan:internal/loomshed#RaddleWriteDeps` -> `type RaddleWriteDeps struct`
- `plan:internal/loomshed#NewRaddleWrite` -> `func NewRaddleWrite(name string, inner shedengine.ShedProducer, deps RaddleWriteDeps) shedengine.ShedProducer`
- `plan:internal/loomshed#TestRaddleWrite` -> `func TestRaddleWrite(t *testing.T)`

**Extends:**
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `internal/loomengine/status.go`

**Uses:**
- `internal/loomengine#DiscussionTable`
- `internal/loomengine#discussionRole`
- `internal/loomengine#Config`
- `internal/loomshed#NewDiscussionWrite`
- `plan:internal/loomshed#UpdateRaddleState`
- `plan:internal/loomshed#ReadRaddleState`
- `internal/seatengine#Table`
- `contracts/specs/raddle-proposal-spec.md`
- `contracts/stencils#TestRegistry_MatchesOnDiskTree`
- `internal/loomshed/doc.go`

**Intent:** D4 and D7's Raddle-Write row, short of its registry entry (card 23).
The stencil `raddle-template-agent.md` is a cold role of its own; it points at `{{.specs_dir}}/raddle/raddle-proposal-spec.md` and restates none of it, and tells the agent to:
- read the run's decision record (the Sketch) at `{{.decision_record_path}}`, `lym board list` and the entries touching the Sketch's packages or module labels, `lym loom peers`, `lym report cost` (JSON) and `lym report timing`, and only the code needed to place each part: the packages and files the Sketch names, which packages each part changes, roughly how much, and who calls whom;
- design nothing and ask the operator nothing until the proposal passes D6's checks;
- weigh that a tapestry's large fixed start cost often makes fewer, larger parts or one tapestry cheaper, with no cost figure written in the stencil;
- write each cost line so it says when a cell it rests on reads "too few runs", never guessing a figure for it;
- in the `{{.mode_rules}}` block Go fills: auto mode, end on the proposal file once it is complete, no operator present; interactive mode, run `lym loom raddle-propose` once the proposal is complete, then put it to the operator, revise the file while the operator questions or corrects the split, order, recipes or profiles, and run `lym loom raddle-approve` only on the operator's explicit approval, never without it;
- when `{{.return_note}}` is set, start from the set-aside proposal at its path and the reason the filing returned it.
`stencils.go` embeds it as `RaddleTemplateAgent`, registers `raddle-template-agent` in `entries`, and maps the `raddle` role to it in `roleOpeningStencils`.
`raddleRole` is the role's one constant (`PATTERN-agent-name`), the table's role prefix.
`RaddleTable` builds a one-seat table like `DiscussionTable`'s lead: model from `cfg.Raddle` under `ModelKey{Module: loom, Key: "raddle"}` (D4's own key, apart from `discussion`), role prefix `raddleRole`, the stencil above, `Interactive` from the recorded mode, its output the approval record in interactive mode and the proposal in auto mode, values `slug`, `specs_dir`, `decision_record_path`, `proposal_path`, `mode_rules`, `return_note` (the reason and the set-aside proposal's path, empty on a first session) and the subagent model; it carries no gate, which the entry sets.
`RaddleWriteDeps` carries the state paths, `AutoApprove func() (bool, error)` (the loom key `raddle_auto_approve`, read through the run's setting lookup so a profile's raddle block may set it), `FreshEntry func() (bool, error)` (true when the row's current attempt began with a Raddle-File return or an operator goto into the row, read from the run's status history, with the history position of that entry), the proposal, set-aside, cold and approval paths, `StoreCold func() error`, which copies the proposal to the cold path and sets `cold_stored` unless already stored (the `raddle` gate's `onPass` in auto mode, a no-op in interactive mode, where `raddle-propose` stores it), and `Commit func() error`, which commits the raddle directory.
`NewRaddleWrite` wraps the table-driven inner producer; each step below is one state update, so a crash-resume repeats none of it:
- At the run's first call, the state's mode empty, it reads `AutoApprove` once and records the mode; every later call and Raddle-File read the recorded mode, never the key, so a key changed after the first spawn, such as by a local-layer config write in the pair, changes no mode of the run; an `AutoApprove` read error is a returned error naming the way forward, re-step the Raddle-Write row, and records nothing.
- Every call raises `session`, so each agent session of the row, a resume after a died session and a halt's resume included, is a session of its own: `raddle-approve` needs a passing `raddle-propose` in it and its first pass posts the waiting notice again (D7); a resume that re-attaches a live session asks the agent for one more propose, which the approve refusal names.
- On a fresh entry not yet taken up at that history position, it clears the last attempt's output so the seat starts a new session rather than finishing on a stale file (`Spec.validate` refuses an output file that already exists): in auto mode it moves the proposal to the set-aside path, in interactive mode it removes the approval record and clears the state's `approval`; it marks a return taken and records the position.
  The prior session ended Done when its row moved on, so its strand is gone and the inner producer spawns a fresh seat; after a goto the operator's goto has stopped the row's strand.
- It then runs the inner producer and, on Done, commits the raddle directory, so a cold proposal or approval is committed under `_lym/raddle/`.
In auto mode the row's `raddle` gate stores the proposal as the cold proposal on the run's first pass (its `onPass`, a no-op once `cold_stored`), so a later pass leaves it, and the last passing proposal is the approved one.
`TestRaddleTable` (tier0) asserts the seat's model key `raddle`, the role prefix, the stencil, the output per mode, the interactive flag per mode and the return note.
`TestRaddleWrite` (tier0, a stub inner producer and temp state) has rows: first call with the key off (mode interactive recorded); first call with the key on (auto); a later call after the key changed (mode unchanged); an `AutoApprove` error (returned, nothing recorded); each call raising `session`; a return in auto mode (proposal set aside); a return in interactive mode (approval removed, state approval cleared); a goto entry with an approval record present (removed); a resume of that new session at the same history position (nothing cleared again); Done commits; an inner error is returned without a commit.
The stencil's static prose takes no test of its own; the registry tests cover its registration.
`loomengine`'s package comment, which sits in `status.go` as the package has no `doc.go`, names the raddle table beside the discussion table.

**ImpactSummary:** A new stencil, its two registry entries and new loomengine and loomshed members; nothing builds the row until card 23.
