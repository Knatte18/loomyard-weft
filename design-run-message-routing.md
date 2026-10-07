# Who in a running task receives an instruction

`run-message-routing` · Note · feature, loom

Who in a running task receives an instruction from the orch or the operator, open for a brainstorm (operator, 2026-10-07).

Seen 2026-10-07: an operator decision about card 1 of `bugfix-sessions` arrived while Webster ran.
The orch has to know which agent owns the content at that moment (Discussion-Write, the plan agent, the Webster master, a Burler round) and message it, or fall back on a goto; mid-Webster no agent can change a begun card at all (`webster-amend-begun-card`).
The orch editing the run's files itself is never the answer.

Options raised:

- The driver forwards: it knows its current producer and the strand doing the work. One address per run, but an LLM relays in a turn of its own, may reword, and its stencil does not do this today.
- An alias such as `ly:<slug>:active` for the strand that owns the current step: deterministic, no LLM between.
  Creel must not know what runs or what is active (operator): it owns messages and injects only.
  So creel could at most store a plain name-to-name alias that loom itself sets at every producer transition, knowing nothing of its meaning; or the alias lives outside creel altogether.
- An on-call LLM per run, woken when an instruction needs judgment about the run (what a decision means for the plan, who must do what). The operator considered this before the driver existed, when the driver was a Go process; it rebuilds its context each time it wakes, and it adds a role.
- The driver itself becomes that on-call role, with read access to the plan: it already lives the whole run, but grows heavier than a step driver.

Plain routing ("deliver to whoever works now") and judgment ("what does this mean for the run") may want different answers.

Steps with no LLM at all: the Go producers (Preflight, Batchifier, the verify gates, Publish, Finalize, teardown) have no strand to deliver to.
A message could wait in its queue for the next LLM producer if the alias is resolved at delivery rather than at send, but that agent may not be the right one (after Batchifier comes the Webster master, while the instruction may concern the plan).
This suggests a split to discuss:
- Content ("the settle comes from config") goes into the decision record, which is durable, read by every later LLM producer, and needs no live receiver.
- Only an instruction that must act now needs a live receiver, and the most urgent one, stop, is a run pause (`lyx batten pause`), which works during a Go step too.

Operator leaning (2026-10-07): the driver takes the message.
It runs a Sonnet agent on purpose, to have judgment; it is the one LLM alive for the whole run, Go steps included, it knows where the run stands, and it receives messages while its `shed step` runs in the background.
With a message it could record content with `lyx loom decision add` (a lyx verb, not a source edit), forward it to the strand that owns the current step, or pause the run or ask for a goto.

Refined (operator, 2026-10-07): every instruction for a run is posted to its driver, which gets only the short reminder (sender and subject) and does not read the body by default.
Knowing which agent works now, it either reads it itself or forwards it with `lyx creel forward` (see the `creel` note), so the receiving agent talks to the original sender, not to the driver: a minimal routing job.
