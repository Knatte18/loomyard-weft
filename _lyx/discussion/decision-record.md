# Decision record — MultiLLMProducer: one producer runs a chair and its advisors as seats

## Goal

`MultiLLMProducer` is a Shed producer that runs N agents in one step, each in a strand of its own.
One seat is the `chair`: its turn end, outputs and gates decide the step's outcome exactly as a single agent's do today, and a message to the producer is a message to the chair.
Every other seat is an advisor, `advisor-1` to `advisor-n`, whose outputs are the chair's inputs and whose questions and answers reach the chair over a channel the parent never sees.
Shed sees one step with one outcome; the seats are invisible to it.
The producer is built standalone and proven by its own tests and a test recipe.
No loom recipe row, no Burler and no Discussion uses it in this task.

## Scope

In:

- A seat runner package, `internal/seatengine`, that starts, joins, probes, resumes and stops the seats of one table over shuttle.
- `shedadapters.MultiLLMProducer`, beside `SingleLLMProducer`, mapping the chair's result onto the Shed producer contract.
- A `MultiLLM` registry entry in `internal/shedrecipe` whose row config carries the seat table, and the `Env` seam it needs.
- A stencil include form, so a seat's one stencil pulls shared building blocks in by name.
- Two shipped shared blocks, `seat-directive-chair` and `seat-directive-advisor`, under `contracts/stencils/seat/`, carrying the channel rules.
- One shuttle `Spec` knob that turns the held-turn parent notice off for advisor seats.
- One send lock per shuttle `Run`, so the in-process sends on one run never type at once (D7).
- A test recipe fixture that builds and runs a chair-and-advisor row against fakes, plus one `llm`-tier smoke run by hand.
- Package docs, the `docs/overview.md` module table and execution stack.

Out:

- Any change to a loom recipe row, to `SingleLLMProducer`, to Burler's round or to Discussion-Write: `burler-multi-llm`, `discussion-multi-llm` and `plan-multi-llm` are their own entries.
- `creel`: the channel here is Claude Code's session-to-session message, as the parent directive already uses; `creel` replaces it later on its own.
- Reed layout changes: `reed-layout-shares` is its own entry.
- A `lyx` verb of any kind: the producer is reached through a recipe row and a Go table only, so no sandbox suite entry applies.
- A new PATTERN entry: every decision below complies with the existing entries, see Constraints.

## Decisions

### D1. Layering: a seat runner below the adapter

**Decision.** The seat-running logic, the table type, its validation, start order, join, failure rules, probe and resume, lives in a new package `internal/seatengine`, imported by `internal/shedadapters`.
`shedadapters.MultiLLMProducer` is the thin adapter the board names: it holds a table and a runner, and maps the chair's result onto Done, Stuck or an error the way `SingleLLMProducer` maps one shuttle result.
`seatengine` imports `shuttleengine`, `stencil`, `stencilstore`, `agentname`, `parentdirective`, `editdirective`, `segmentcolor`, `logger` and the standard library, and never `lyxcwd`, `burlerengine`, `shedadapters` or any `*cli` package.

**Rationale.** `shedadapters` is by its own package doc a translation layer over shipped engines, with Bouncer the one exception.
Starting N strands, joining them, stopping the rest on a decision and attaching to a live set on resume is engine logic of burler's `Run`/`join`/`ProbeRound`/`Resume` kind.
`burler-multi-llm` replaces burler's `roundHalves` with a seat table; `burlerengine` can import a package below it but not `shedadapters`, which imports `burlerengine`.
The producer itself stays in `shedadapters` as the board says.

### D2. The seat table is the one declaration, in Go, with a recipe form

**Decision.** `seatengine.Table` is the one place a multi-seat step's agents are declared: a role prefix, a segment color, an optional gate for the chair, an optional timeout, an interactive flag for the chair, shared stencil values, and an ordered list of `Seat` entries.
A `Seat` carries its name (`chair` or `advisor-<n>`), exactly one stencil name, the resolved model triple shuttle takes (model, effort, version), its inputs (absolute file paths it is given), its outputs (absolute files it writes, at least one), the skills lyx types into it, and per-seat stencil values.
The recipe entry `MultiLLM` builds the same table from a row config with keys `role`, `segment`, `seats`, and optional `gates`, `timeout_min`, `interactive`, `tokens`; each `seats` element carries `name`, `stencil`, `model`, `inputs`, `outputs`, and optional `skills`, `values`.
A seat's `model` is model-spec notation (`opus[high]`), resolved through `modelspec` against the registry the `Env` carries, so a recipe row never spells a provider id.
Inputs and outputs are worktree-relative in the recipe and resolved under `Env.WorktreeRoot`, as the `SingleLLM` row does.

Validation at construction, every rule a loud error naming the seat:

- exactly one seat named `chair`;
- every other seat named `advisor-<n>` with `n` running from 1 contiguously in table order;
- every seat has at least one output, and no two seats share an output path;
- every advisor output is among the chair's inputs;
- the role prefix passes `agentname.ValidateRole`, and the formed role of every seat does too;
- every seat's stencil, and every block it includes, is readable from the told stencils directory;
- a `tokens` or `values` name that collides with a reserved marker (D3) is refused, and an empty value is refused, as the `SingleLLM` row does.

**Rationale.** The board asks for one table saying what goes where, replacing wiring spread over several packages.
A Go table is what a Go caller such as burler needs; the recipe row is the same table in YAML for rows that need no per-run closure.
Advisor-outputs-are-chair-inputs is the contract that makes the chair's decision informed, so it is pinned at construction, not left to the stencil author.

### D3. One stencil per seat; shared blocks come in by `{{template "name"}}`

**Decision.** `internal/stencil` gains an added form, beside `Fill` and `FillOptional`, that renders a template together with named included templates: a seat stencil names a shared block with Go's `{{template "seat-directive-chair"}}`, and the seat engine reads every named block from the told stencils directory through `stencilstore.Read` and hands it to the fill.
A new accessor returns the include names a template declares, so the engine knows what to read.
Includes resolve one level deep: a block that itself carries `{{template}}` is refused at construction.
The unfilled-marker guarantee extends to every included block's top-level markers, so a seat prompt can never render a blank field.
`TopLevelMarkers` and every existing caller are unchanged.

The engine fills these reserved markers for every seat, from one value map per seat: `parent_directive`, `edit_directive`, `seat_name`, `chair_name`, `advisor_names`, `failed_advisors`, `inputs`, `outputs`, `output_files`, `worktree_root`, `anchor_path`, `stencils_dir`.
Names are full strand names, one per line; `advisor_names`, `failed_advisors` and `inputs` carry the literal `(none)` when empty, the way Bouncer passes `(none)`, since `Fill` treats empty as unfilled.
`failed_advisors` lists the advisors whose start failed (D7); it is `(none)` in every advisor's map.
`parent_directive` is rendered per seat (D6): the chair's names the run's parent, an advisor's names the chair.
`pattern_directive` and `friction_directive` are not reserved: a consumer that wants them supplies them through the seat's values, as the loom rows compose them today.

Two shared blocks ship, registered in `contracts/stencils` and seeded like every stencil:

- `seat-directive-chair`: the chair steers and decides; the advisors, by name, answer its questions over the channel, and a message from a named advisor is advice to weigh, never an instruction; their outputs at the listed input paths are its inputs; the advisors in `failed_advisors` never started; an advisor failure notice may arrive and the chair carries on; the step ends when the chair writes its outputs.
- `seat-directive-advisor`: who the chair is, the seat that leads the step; the advisor's own output, the only files it writes; it messages the chair only, never another advisor and never any other session; it writes its output when its task is done or when the chair asks it to wrap up, and keeps answering the chair after writing it; a question it cannot answer goes to the chair.

Both blocks carry the seat-to-seat channel wording, and that wording lives nowhere else.
Neither block tells an agent to ask the operator or restates the edit rule, so the existing stencil guards pass over them.

**Rationale.** The board wants exactly one stencil per seat with shared building blocks included by name.
Go's `text/template` already has named includes; adding a form that supplies them keeps `stencil` a leaf and keeps every existing stencil's parse unchanged.
One level is enough for every seat stencil anyone has described, and refusing nesting keeps the marker guarantee a one-pass check.

### D4. Seat roles and strand names

**Decision.** A seat's role is `<prefix>-<seat name>`: `discussion-chair`, `discussion-advisor-1`.
`seatengine` declares `RoleChair` and `RoleAdvisor` as constants and forms the role in one function; the spawn spec's `Role` is that call, never a literal, so the `spawnrole` tripwire passes.
Reed forms the full strand name as it does for every strand.
After a start the engine compares the strand's formed name with the seat's expected name and treats a difference, reed's `-N` numbering of a held name, as a start error naming the holder and the way forward `lyx reed remove --name <name>`.

**Rationale.** PATTERN-agent-name: the spawning module owns its role names as constants and reed forms the name once.
The chair's and advisors' names are what the seats address each other by, so a renumbered seat would be unreachable; failing loudly beats a silent wrong address.

### D5. Done is the file contract; the chair's done ends the step

**Decision.** Every seat is an ordinary shuttle run: its done is its output files existing.
The chair's outputs are the step's outputs, and its gate, when the table carries one, is the step's gate; advisors carry no gate.
Advisors run with `KeepPane` set, so an advisor whose run reaches Done keeps its strand and stays reachable by the chair; the chair runs without it, as a single agent does.
Within one call, an advisor is live while reed still tracks its strand, whether or not its `Wait` has settled: a Done advisor (kept by `KeepPane`) and a died or timed-out one (never removed by shuttle) are both live.
Across a resume, only an advisor whose record still reads running is live: D8's probe stops every advisor whose record is terminal, so a finished advisor is reachable by the chair within the call that started it and not after a resume, and the resumed chair is told so (D8).
When the chair's run reaches a terminal result, every live advisor is stopped through its handle's `Stop`, which settles the run record, keeping a settled record's outcome, before the strand goes; an advisor's existing output files stay on disk.
A `Stop` that fails is an error naming `lyx reed remove <guid>` and a re-step as the way forward, never retried.
The chair's result maps as `SingleLLMProducer.mapOutcome` does: Done with the chair's first output as the pointer; a failing gate to Stuck with the pointer and the gate reason; died or timeout to an error, wrapping `ErrNotStarted` when the provider never came up.
The seat engine reads shuttle's outcomes and makes no "did it finish" judgment of its own, so PATTERN-completion-signal's check stays inside shuttle.

**Rationale.** The board: the chair's turn end, outputs and gates decide the step's outcome as a single agent's do today.
An advisor's done is a fact for the chair, not for Shed.
A stopped advisor's output stays because the chair, or the next step, may still read it.

### D6. The channel is Claude Code's session message, by strand name, chair-centred

**Decision.** Seats message each other with Claude Code's `SendMessage`, addressed by full strand name, exactly as the parent directive and the orch role already do; `ListAgents` shows the names.
This is the operator's decision (2026-10-10): `SendMessage` is the seats' channel until `creel` is up and replaces it.
Neither `lyx shuttle send` nor shuttle's `Run.Send` is ever the seats' channel; the engine's one Go-side use of `Run.Send` is D7's notice to the chair.
The names a seat may address are in its prompt through the reserved markers.
Advisors message the chair only; the chair relays what another advisor must hear.
An advisor's `parent_directive` is rendered with the chair's strand name as the parent name, so the only session an advisor's prompt names as its parent, to ask and to trust, is the chair; the run's parent appears in no advisor prompt.
The chair's `parent_directive` names the run's parent as a single agent's does, and the chair block makes a named advisor's message advice to weigh, never an instruction.
A message to the producer is a message to the chair: the chair's strand name is the producer's address, and `lyx shuttle send <guid>` remains the typed path for Go and the operator.
`seatengine` adds no provider words; the seat-to-seat channel's words live in the two seat directive stencils only.

Bound: the "advisors message the chair only" rule is prompt-enforced and caught by review; no mechanism scopes a session's message targets, so an advisor that disobeys can reach any name `ListAgents` shows, the run's parent included.
What bounds it: no advisor prompt names any session but the chair and its fellow advisors, advisors are never interactive (D10), and their held turns notify no parent (D10).
An advisor's message to the chair carries no delegate authority: the chair weighs it as advice.

Bound on the chair-to-advisor edge: the parent directive makes a chair message an operator-grade instruction to the advisor, within the advisor's role, so the chair can pass on as an instruction what another advisor wrote to it as advice.
What it can let through is an instruction to the advisor; what bounds it is the advisor's role and settings, never the chair's word: the advisor block confines its writes to its own output files, its agent settings' denies hold, and it is never interactive (D10), so the most a relayed instruction can cost is a wrong or wasted advisor output, which the chair weighs as advice in turn.
The writes limit is prompt-enforced like the message-target rule above.
The directive's clause that the parent is "the session that spawned this worktree's run" is false for the chair and is accepted as rendered: the advisor acts on nothing in it but whom to ask and trust, and the advisor block states the chair's real role, the seat that leads the step.

The channel carries nothing a seat does not write into a file, so a dropped message costs a repeat, never the step.

**Rationale.** A message delivered into a running turn reaches the chair at its next model call, so the chair asks and keeps drafting instead of ending its turn to wait, which is what `discussion-multi-llm` describes.
Naming the chair as an advisor's parent reuses the parent directive's ask-and-trust rule for the one session an advisor answers to, instead of a second wording of it.
The weakness creel's note records, a message lost across a `/clear`, does not arise inside one step.

### D7. Failure and hang rules

**Decision.** The rules, closed:

- Chair fails to start: every advisor already started is stopped; the error is returned, wrapping `ErrNotStarted` when the provider never came up.
- Advisor fails to start: the step goes on; the seat's `StartError` is recorded on the step result.
  Advisors start before the chair (D8), so the chair is told once through its prompt: the failed advisor's name is in the chair's `failed_advisors` marker (D3), and no `Send` is made for it.
- Advisor dies or times out mid-step, or is not found on resume with the chair live (D8): the step goes on; the chair is told once by a Go-side notice.
- Advisor reaches done: nothing happens; it stays reachable (D5) and the chair reads the file.
- Chair reaches any terminal result: every live advisor is stopped (D5).
- One `Timeout` governs every seat; it is the table's, else shuttle's run default.
  Advisors start first, so an advisor still without its output times out just before the chair's deadline; that is the routine end of an advisor the chair never asked to wrap up, and it is told like any other timeout.
- Context cancellation returns the context error, never Stuck, and leaves live seats for the next call's probe (D8), as `SingleLLMProducer` does.

Each advisor's `Wait` is joined from the moment that advisor starts, so an advisor can end while later seats, the chair included, are still starting.
The Go-side notice is one line typed by `Run.Send` on the chair's handle, from one sender goroutine per chair that drains a queue one line at a time, separate from the goroutines that join the seats' `Wait`s, so a send never blocks the join and two notices never type at once.
An advisor end observed before the chair's handle exists is queued like any other; the sender starts when the chair's start returns its handle, and a chair that fails to start drops the queue with the rest of the step.
`Run.Send` waits for an idle session and fails with `ErrSessionBusy` past its send-ready window, so the notice reaches the chair only at an idle turn boundary, a turn the chair ends without its outputs.
The sender retries a busy or unlanded line until it lands, all the chair's outputs exist or the chair's run settles, and stops at the first of the three; a notice that never lands is logged, not an error.
A chair with its outputs written has decided, so a later notice would change nothing; the outputs check runs before each `Run.Send`, so it cannot stop a send already in its idle wait.
That send can meet the chair's own `Wait` at the gated Done boundary, where the gate types its re-prompt or pending send.
Shuttle gains one send lock per `Run`, held across the whole idle-wait, typing and confirm sequence by `Run.Send` and by the gate's send, so the notice and the gate's line type one after the other into the pane, never at once.
Bound: the lock only orders in-process sends on one run; it skips no idle wait, no delivery check and no gate evaluation, and a send waits on it no longer than the holder's send-ready and submit windows.
Whichever line goes second meets the turn the first one started: the gate's existing turn-start hold keeps its send off that turn as it does for any send at a boundary, and a notice that loses the race waits for idle or fails busy and is retried.
`lyx shuttle send` on the chair's guid runs in another process and is outside the lock, as it is beside a single agent's gate sends today: a collision with it garbles or fails one line, which the notice sender's retry, the gate's busy and unlanded re-send and the verb's own failure report each recover.
The trade-off accepted: a chair that never idles before writing its outputs never hears the notice, and learns of the advisor's loss only from its silence, which the hang rule below covers.

An advisor hang is not detected by the engine: an idle advisor waiting for the chair's questions is the normal state, so idle without output is not a stall for an advisor.
The chair's directive gives it the rule: an advisor that does not answer in reasonable time is written off and the chair proceeds.
Shuttle's session-state model is read by nothing here; it stays in shadow mode as `shuttle-state` decided.

Bound: an advisor's death weakens nothing the chair decides; the step cannot end Done because an advisor died, only because the chair wrote its outputs.

**Rationale.** The chair owns the outcome, so an advisor's loss is information for the chair, not a verdict.
Telling the chair through shuttle's own `Send` keeps the notice provider-neutral and lets the chair decide.
A start failure is known before the chair's prompt is filled, so the prompt carries it exactly once with no send to retry.
An advisor that ends during the chair's start is queued, not added to `failed_advisors`, so the marker stays a fixed fact of the start order and every end after a start takes one path.

### D8. Start order, probe and resume

**Decision.** On every call the engine probes before it archives anything: the chair with the superseded-removing gated probe, each advisor with the same probe shape ungated.
An advisor whose record is terminal and whose strand is still up, a Done advisor kept by `KeepPane` or a timed-out one, is respawn-eligible to that probe, which stops its strand, keeping the record's outcome, and answers not found.
If the chair is live, the call attaches to it and joins every advisor the probe found; an advisor not found is not restarted, its seat is gone, and the chair is told once by D7's Go-side notice, the chair's handle being the probe's.
The notice is worded by the advisor's outputs: an advisor whose outputs all exist is finished, and its line says the output stands and the seat no longer answers;
any other is ended, and its line says so as a mid-step death does.
A notice sent before the interruption may land a second time; a repeated line costs the chair nothing.
If the chair is not live, every live advisor is stopped, every seat's stale outputs are archived with the shared archive helper, and the seats start fresh: advisors in table order, the chair last.
A fresh start writes nothing durable of its own; each seat's state is its shuttle run directory.

**Rationale.** Probe-before-archive is the hazard `SingleLLMProducer` documents; the same ordering applies per seat.
Shuttle's probes hand back no handle for a terminal record, so the removing probe is the one path that leaves no advisor strand without a handle to stop it; a finished advisor's output is on disk.
The advisor block tells the chair a finished advisor keeps answering, so its removal is told like any other end, and the chair does not spend the hang rule's wait on a question to a seat that is gone.
A chair that is gone has lost the dialogue its advisors were part of.
Chair last is the layout decision (D9) and costs nothing for addressing, since names are deterministic and advisors begin by studying, not by messaging.

### D9. Layout until `reed-layout-shares`

**Decision.** The seats take reed's one layout rule as it is: insertion order, the newest strand expanded, the rest collapsed.
Advisors start first and the chair last, so the chair is the expanded pane and the advisors are collapsed rows above it.
Every seat's display is below-parent with no parent set, as the loom rows spawn today; no layout API changes.

**Rationale.** The board leaves layout open until `reed-layout-shares`; the operator reads the chair, so the chair gets the space the current rule can give.

### D10. The chair alone may be interactive; advisors' held turns are quiet

**Decision.** The table's `interactive` flag applies to the chair only; advisors always run autonomous.
Shuttle's `Spec` gains one knob, set by `seatengine` for advisor seats only, that skips the held-turn parent notice.
Bound: the knob skips the orch notice alone; the hold itself, the wait's polling, the deadline, the wait mark and attach behaviour are unchanged, and nothing else reads it.
The chair's held turn notifies the parent as a single agent's does today.

**Rationale.** An advisor waiting for the chair's next question ends turns without its output as a matter of course; one orch notice per such turn is noise.
A chair that ends a turn without its outputs may be stuck, and the parent should hear it as it does today.
An interactive advisor would take questions from a pane the operator does not watch.

### D11. `SingleLLMProducer` stays; N = 1 is a seat table with a chair alone

**Decision.** `SingleLLMProducer` is unchanged.
`seatengine` admits a table of one chair, and a test pins that such a table maps every shuttle outcome as `SingleLLMProducer` does.

**Rationale.** `SingleLLMProducer`'s seam is result-shaped and shared with Bouncer and every test fake, and no consumer of the seat engine exists yet.
The N = 1 claim is true by construction and pinned by a test, which is all the board asks of it.

### D12. The recipe entry's seam

**Decision.** `shedrecipe.Env` gains one seam, a seat runner over the caller's shuttle runner, nil when no row uses it.
A `MultiLLM` row with the seam nil is a construction error, the rule `commit_seam` follows.
Loom's `Env` builder fills the seam, so a loom row may name the engine later; none does in this task.

**Rationale.** `Env.Shuttle` is the result-shaped `shedadapters.Shuttle`; seats need the handle-shaped seam burler has (`StartGated`, `ProbeGated`, a handle with `Wait`, `Stop`, `Send`, `StrandGUID`, `RunDir`), which `*shuttleengine.Run` already satisfies.
A present key with an absent seam must fail loudly, never run with nothing.

### D13. Test strategy and tiers

**Decision.** Tier 1 throughout, with fakes: a handle-shaped fake shuttle for `seatengine`, a fake seat runner for `shedadapters`, the existing conformance tables for `shedrecipe`.
The test recipe is a fixture recipe with one `MultiLLM` row of a chair and two advisors, built through `shedbuild` with fixture seat stencils written into a seeded temporary stencils directory under an unregistered prefix, which `stencilstore` resolves by its first dash segment without registration.
One `llm`-tier smoke in `seatengine` runs a real chair-and-advisor pair from those fixtures: the advisor answers one question over the channel, the chair writes its output, the step ends Done, the advisor is stopped.
`seatengine` gets a `TestMain` through `tmuxkit.Main` because it carries an `llm` file.
`cmd/lyx/testdata/test-budget.yaml` gains a row for `internal/seatengine` and raises `internal/shedadapters`, which sits at its budget; `internal/shedrecipe` and `internal/stencil` are checked at Plan time.

**Rationale.** PATTERN-test-speed: the lowest tier the substrate needs; only `llm` files reach an LLM.
PATTERN-test-economy: behaviour at the public surface, extending the existing `SingleLLM` and burler tables where a scenario is shared.

### D14. Docs in the same commit

**Decision.** `internal/seatengine/doc.go` states the seat model and every rule above; `internal/shedadapters/doc.go` adds the fifth adapter and its outcome mapping; `internal/shedrecipe/doc.go` and `contracts/specs/shed-recipe-spec.md` describe the `MultiLLM` row and its seam; `internal/shuttleengine/doc.go` states the quiet-hold knob and the per-run send lock; `docs/overview.md` adds `seatengine` to the module table and the execution stack and corrects the adapter count.
No PATTERN entry changes.

## Constraints

- PATTERN-told-geometry and PATTERN-cwd-resolution: `seatengine` is handed absolute roots and derives none.
- PATTERN-shed-recipe-registry: one map entry `MultiLLM`, no `init()`.
- PATTERN-shed-producer-seam: `shedengine` is untouched.
- PATTERN-agent-name: role constants in the spawning module, names formed by reed (D4).
- PATTERN-stencil-ownership: every seat stencil and block is read at call time from the told directory, with an eager existence probe at construction as the `SingleLLM` row does.
- PATTERN-parent-directive and PATTERN-edit-directive: a consumer's seat stencil is an opening stencil and renders both markers; the engine fills them; the two shipped blocks tell no agent to ask the operator and restate no edit rule.
- PATTERN-role-skills-typed and PATTERN-spawn-color: skills and segment ride the spec; no stencil asks for either.
- PATTERN-shuttle-provider-seam: `seatengine` adds no provider words; the seat channel's words live in the seat blocks (D6).
- PATTERN-shuttle-stop: seats are stopped through their run handles, which settle the record first.
- PATTERN-completion-signal: the engine decides nothing about finishing beyond shuttle's results (D5).
- PATTERN-transient-stop: the engine marks nothing transient; its hard errors reach the shed classifier as errors.
- PATTERN-test-speed, PATTERN-test-isolation, PATTERN-testkit: D13.
- PATTERN-sandbox-coverage: no new module is registered, so no suite entry.
- PATTERN-comment-line-breaks: every new Go comment one sentence per line.
- No PATTERN entry is changed or waived.

## Auto-mode assumptions

- A1. The channel is Claude Code's `SendMessage` (D6), read as the board's "own internal channel" in the sense that the parent sees none of it and no lyx module routes it.
- A2. Advisors do not message each other (D6).
- A3. An advisor's death or hang never fails the step (D7).
- A4. `SingleLLMProducer` stays as it is (D11).
- A5. The runner lives in `internal/seatengine`, the producer in `internal/shedadapters` (D1).
- A6. Advisors start first, the chair last (D8, D9).
- A7. Only advisors' held-turn notices are suppressed (D10).
- A8. Stencil includes are one level deep (D3).

## Open risks

- The channel's provider dependence: `SendMessage` is Claude Code's, and a second engine would need its own channel words in the seat blocks; creel removes the dependence later.
- An advisor hang is invisible to the engine (D7); the chair's directive is the only bound, and a chair that waits anyway shows as a held turn to the parent.
- An advisor death notice lands only when the chair idles (D7); a chair that drafts straight through to its outputs never hears it.
- An advisor that disobeys its block can message any strand `ListAgents` shows (D6); only review catches it.
- The chair can relay one advisor's advice to another as an instruction (D6); the advisor's role, its settings' denies and the chair's own weighing of the result bound it.
- A finished advisor is stopped by a resume's probe (D8), so a chair resumed mid-dialogue cannot ask it again; the chair is told, and the advisor's output stands.
- A chair that never ends its turn to wait has no way to receive a message between model calls other than the provider's delivery; if delivery into a running turn proves unreliable in the smoke run, the fallback is the chair's directive telling it to end its turn after asking, which the hold notice path already supports.
- Reed's layout collapses advisors to a few rows until `reed-layout-shares`; the operator zooms a strand to read it.
- The stencil include form is a change to a leaf every stencil passes through; the guarantee that every existing stencil parses and fills identically is pinned by the existing marker and claims guards.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - notice-stop-not-off-gate-boundary (class design, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

Table and construction:

- A table with no chair, two chairs, an advisor numbered out of order or from 2, a seat with no output, two seats sharing an output, or an advisor output missing from the chair's inputs is refused with an error naming the seat.
- A reserved marker name in `tokens` or `values`, an empty value, an unreadable stencil, an unresolvable include and a nested include are each refused at construction.
- The recipe entry rejects an unknown key at the row and at the seat level, and a `MultiLLM` row with a nil seat seam.
- A seat `model` in model-spec notation resolves to the spec's model, effort and version; an unknown alias is a construction error.

Spec composition:

- Every seat's role is `<prefix>-<name>`; its segment, skills, timeout and the chair's interactive flag ride its spec; advisors are never interactive and carry the quiet-hold knob; the chair does not.
- Every reserved marker is filled in every seat; `advisor_names`, `failed_advisors` and `inputs` read `(none)` when empty.
- An advisor's parent directive names the chair's strand name; the chair's names the run's parent.
- Advisors' specs carry `KeepPane`; the chair's does not.
- A seat stencil including `seat-directive-chair` or `seat-directive-advisor` renders the block's markers from the seat's value map.

Running:

- Advisors start in table order and the chair last.
- Chair Done maps to Done with the chair's first output as the pointer; live advisors are stopped.
- A failing chair gate maps to Stuck with the pointer and the gate's reason; advisors are stopped.
- Chair died or timeout returns an error; `ErrNotStarted` is wrapped when the provider never came up; advisors are stopped.
- A chair that fails to start stops every started advisor and returns the error.
- An advisor that fails to start leaves the step running, is named in the chair's `failed_advisors` and is recorded on the result, and no notice is sent for it.
- An advisor that dies or times out leaves the step running, puts exactly one notice line to the chair once the chair is idle, and is recorded on the result; a busy chair delays the notice without blocking the join, and a chair that settles first, or whose outputs all exist first, ends the retries.
- Two advisors that end together put their two lines to the chair one after the other, never interleaved.
- An advisor that ends while the chair's last turn writes its outputs, with the notice already waiting for idle, never types at the same time as the gate's re-prompt or pending send at that boundary; whichever goes second waits for idle, and the gate spends no attempt on it.
- An advisor that ends while the chair is still starting has its notice sent once the chair's handle exists; a chair that fails to start sends nothing.
- Within one call, an advisor that reaches Done keeps its strand until the chair's terminal result stops it; a died or timed-out advisor's strand is stopped then too.
- A failed `Stop` returns an error naming the way forward and is not retried.
- Cancellation returns the context error and stops nothing.
- A one-chair table maps every outcome as `SingleLLMProducer`'s table does.

Probe and resume:

- A live chair is attached, advisors whose record reads running are joined, an ended advisor is not restarted and the chair is told once.
- With the chair live, an advisor whose record is Done and whose strand was kept has its strand stopped by the probe, its outputs left in place, and draws one notice saying its output stands and its seat is gone; a timed-out advisor whose strand was up is stopped and draws one notice saying it ended.
- A chair that is not live has its live advisors stopped and every seat's stale outputs archived before a fresh start, and nothing is archived while the chair is live.
- A strand holding a seat name after the probe is a start error naming the holder.

Shuttle and stencil:

- The quiet-hold knob skips the parent notice and changes neither the hold, the wait mark nor the outcome.
- Two concurrent `Run.Send` calls, and a `Run.Send` beside the gate's send, on one run type one after the other, and each still waits for idle and verifies delivery.
- The include form renders `{{template}}` blocks, reports an unfilled marker inside a block, and leaves `TopLevelMarkers` and every registered stencil's fill unchanged.
- The two seat blocks are registered, their markers pinned, and pass the ask-operator and edit-rule guards.
- The spawn-role tripwire passes.

Recipe and smoke:

- The fixture recipe builds through `shedbuild` and runs a chair with two advisors to Done against fakes.
- The `llm` smoke (run by hand) sees the advisor answer over the channel, the chair finish, and the advisor stopped.

## Notes for the plan writer

- `internal/burlerengine/engine.go` (`Run`, `join`, `decideRound`, `stopHalf`) and `resume.go` (`ProbeRound`, `Resume`) are the closest precedent for starting several runs, joining them and stopping the rest; `internal/burlerengine/shuttle.go` is the handle-shaped seam shape to copy.
- `internal/shedadapters/singlellm.go` is the outcome mapping and probe-before-archive ordering to mirror; `archive.go` holds the shared archive helper.
- `internal/shedrecipe/entries_singlellm.go` shows the reserved-token rule and the eager stencil probe; `resolveGateSpec` is the `gates` key; `configMapList` reads a list of maps.
- `internal/shuttleengine/hold.go` (`notifyHeld`) is where the quiet-hold knob is read; `run.go` has `Run.Send` and `send.go` the gate's `sendWithin`, the two sends the lock covers.
- `contracts/stencils/stencils.go` holds the registry and `roleOpeningStencils`; the seat blocks are not opening stencils and join the registry only.
- `internal/stencil/stencil.go`: `FillOptional` is the form to add beside; `topLevelMarkerNames` is the walk to extend over included templates.
- `internal/testkit/stencilkit` seeds a stencils directory; `internal/testkit/shuttlefake` fakes the engine and reed for a real `shuttleengine.Runner`.
- `contracts/specs/llm-model-spec.md` and `internal/modelspec` for the seat `model` key; `landingshed.Deps` shows how a registry is told through `Env`.
