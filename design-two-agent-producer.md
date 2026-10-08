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


## From issue #444

Imported from [issue #444](https://github.com/Knatte18/loomyard/issues/444).

**What happened**
Re-stepping Discussion-Burler after the Send failure (see the companion issue on `shuttle: Send` leaving text pending) halted the run as `failed` with anomaly `producer-hard-failure`:
`round 2: probe the round's halves: burler: probe the burler-review half: shuttle: attach: 1 candidate run dir(s) under .lyx/shuttle cannot be confirmed dead or alive (untracked or with a cleared pane binding, younger than 3m0s): .lyx/shuttle/<run-id> — check "lyx reed status" and either wait or clear the stale directory by hand`.
The run history at the halt had 6 entries (Discussion-Bouncer/Burler alternating `stuck`).

**Why it is abnormal**
The candidate run dir is almost certainly the half that `burler: stopping a live half of a round` had just stopped in the previous attempt.
lyx created that state itself, yet the next probe cannot classify it and treats it as a hard failure instead of waiting out the 3m window.
An automated run should not need a manual "wait or clear by hand".

**What to do**
Have the stop path mark the run dir dead (or clear its pane binding explicitly) so the probe sees it as dead.
Alternatively make the probe's "cannot be confirmed" outcome transient, so the shed retries after the grace window rather than halting.

Provenance: task `plan-glyph-precision`; notes: `loom-halt.md`, `driver-repair-20261008-discussion-burler-1.md`.
