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
