# A producer runs two live agents side by side

`two-agent-producer` · Note · feature, shuttle

A producer that runs two live agents side by side in one step, each in a strand of its own (operator, 2026-10-07).
Today every producer runs one agent through shuttle and waits on its one turn end; the Webster master's forks are sub-agents inside one session, not two strands.

Two notes need it:

- `discussion-advisor`: beside Discussion-Write, a second agent explores the task and the code and answers its design questions as they arise.
- `burler-split`: a review-only agent forking its lenses, and in parallel a stronger fixer that orients, then fixes and validates the findings.

What the shared capability has to cover: shuttle starting both agents with their own model-specs, waiting on both and knowing when each is done (the hard part, see `shuttle-crucible`), ending the step when the producer's own rule says so, and the two agents talking to each other directly, which points at `creel`.
Built once, it serves both notes.

Inside one producer, not two parallel shed producers (operator, 2026-10-07): shed is a linear machine, one current producer, one history, and pause, goto and routing each assume one step at a time.
Parallel producers would need fork and join in the recipe and rules for one branch halting while the other is done; inside one producer the two agents are invisible to shed, which sees one step with one outcome.
