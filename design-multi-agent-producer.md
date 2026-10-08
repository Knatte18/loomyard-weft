# A producer runs N agents as seats, one of them the master

`multi-agent-producer` · Note · feature, shuttle

A producer that runs two live agents side by side in one step, each in a strand of its own (operator, 2026-10-07).
Today every producer runs one agent through shuttle and waits on its one turn end; the Webster master's forks are sub-agents inside one session, not two strands.

Two notes need it:

- `discussion-advisor`: beside Discussion-Write, a second agent explores the task and the code and answers its design questions as they arise.
- `burler-split`: a review-only agent forking its lenses, and in parallel a stronger fixer that orients, then fixes and validates the findings.

What the shared capability has to cover: shuttle starting both agents with their own model-specs, waiting on both and knowing when each is done (the hard part, see `shuttle-crucible`), ending the step when the producer's own rule says so, and the two agents talking to each other directly, which points at `creel`.
Built once, it serves both notes.

Inside one producer, not two parallel shed producers (operator, 2026-10-07): shed is a linear machine, one current producer, one history, and pause, goto and routing each assume one step at a time.
Parallel producers would need fork and join in the recipe and rules for one branch halting while the other is done; inside one producer the two agents are invisible to shed, which sees one step with one outcome.


## Generalized: N agents, one master (operator, 2026-10-08)

The single-agent producer and the two-agent producer become one shape: a producer runs N agents, N ≥ 1, of which exactly one is the master.
The master's turn end, outputs and gates decide the step's outcome, as a single agent's do today; the others are helpers whose start, model-spec, wait and teardown the producer manages, and whose results reach the master.
A single-agent producer is the case N = 1.

Burler's round already runs two agents ad hoc since 572ab84e8 (`roundHalves` in `internal/burlerengine/engine.go`): the reviewer, and the fixer, which waits on the review through `lyx burler await-review`.
In the general shape the fixer is the master and the reviewer a helper; `discussion-advisor` is Discussion-Write as master with the advisor as helper.

Open for the task that builds it: what a helper's failure or hang does to the step, how the producer knows a helper is done (the session-state model `shuttle-state` builds), and how master and helpers talk (`creel`).

## Seats: one table says what goes where (operator, 2026-10-08)

Burler's round today wires six stencils (two orchestrator templates, three step files, a focus directive) plus per-segment rubrics and the Bouncer's judge stencils through Go code spread over several packages, the same wiring friction the single-agent producer had, multiplied.
An N-agent producer declares its agents as seats, in one place (the recipe row or one Go table):

- a name and a role: master or helper;
- exactly one stencil per seat, the only prompt that agent gets; shared building blocks such as today's step files are included from it by name;
- a model key in config;
- inputs (files and markers the seat is given) and outputs (files it writes); a helper's output is a master's input, and that is the only coupling between seats, so no agent is told about another in any other way.

A test pins that every seat's stencil markers are filled (`stencil.Fill` already refuses an empty marker) and that every helper output is some master's input.
