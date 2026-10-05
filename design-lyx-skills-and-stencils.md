# The orch role stencil, cycle and distill with a tailored note, ly-drive as a stencil

`lyx-skills-and-stencils` · Task · enhancement, stencil, orch, shed · running · Plan-Burler

Depends on: [`test-writing-rules`](design-test-writing-rules.md)

# Goal

Every role lyx spawns gets its whole procedure from a stencil, the orch included, and lyx itself reloads that procedure and the skills it needs on every path in, so no session depends on an agent remembering rules or choosing to load a skill.

# Decisions (operator, 2026-10-05)

- **Skill or stencil.** A skill carries what any session needs to use lyx, the operator's own sessions included; a stencil carries one spawned role's procedure.
  `ly:board` stays a skill, since any agent may add to the board.
  The planned `ly:runs` skill is dropped: only the orch runs the loop, so the loop goes into the orch's role stencil.
- **Skills are workflow guides, not references.** Cobra's `--help` stays the reference for flags and payload keys; a skill gives the concepts, the order of steps and one example per verb.
- **lyx loads skills itself; it never relies on a hook or an agent's choice.** The scribe plugin's `SessionStart` hook asks a session to load `scribe:conversation` and `scribe:prose`, and on 2026-10-05 the orch saw the request and did not load either.
  The orch watcher types the skills' slash commands (`/scribe:prose`, `/scribe:conversation`, `/ly:board`) into the session itself.
- **The orch does not use `scribe:handoff` to cycle.** That skill is general; the orch needs a short, tailored note, and lyx already knows the rest (runs through `lyx batten status`, the board through `lyx board`).

## The orch role stencil

One committed stencil under `contracts/stencils/orch/` carries the orch's whole working procedure, so an orch on any machine, or after any cycle, works by the same rules; today they sit in one machine's file-memory and in handoffs.
It absorbs the run-loop content planned for `ly:runs` and the role section of note `orch-owns-the-loop`, one section per theme:

- The run loop: spawn a task with `lyx batten run <slug>` from the prime, detached; read progress with `lyx batten status`; use `pause` and `goto`; batten notices (`[batten notice]`) wake the orch, so it starts no polling shell or Monitor.
- Parent-review: a message naming a brief goes to a one-shot fork that reads the brief, checks scope against the board entry only, and submits with the brief's `lyx loom review` command; a repeat notice starts no second fork.
- Escalations from a child (CIRCLING, a spent review budget): `lyx loom circling continue|accept`, or `lyx shed goto` for a fresh budget, then tell the driver to resume.
- PR-Gate: the orch approves or rejects itself after a one-shot reviewer fork, judging the run against its own board goal and its tests, never widening scope through rejects; it fixes a small goal defect on the task branch and puts new hardening on the board instead of rejecting.
- Landing and after: what the orch does once a run lands, such as amending board entries the run superseded.
- Board: read and write only through `lyx board`; a proposal not yet shaped into a task goes on the board as a note at once; fewer, larger tasks; a small finding folds into an open entry whose work overlaps.
- Bugs in lyx go to GitHub issues; design, features and hardening go on the board.
- Act and fix without asking; ask the operator only about what to build.
  An instruction from the operator to a strand is relayed in the operator's own words and scope.
- Every status report on a run gives its `cd <pair> && lyx reed attach` command.
- Every lyx agent is reachable by name through Claude Code's `ListAgents` and `SendMessage`, under `internal/agentname`'s form (`ly:orch`, `ly:<slug>:driver`, `ly:<slug>:discussion`), so the orch messages a run's driver or writer directly; it never types into a pane.

`lyx orch start`, `--adopt` and every cycle render this stencil into the session.
The matching file-memory entries on the hub machine are deleted when the stencil lands.

## Cycling: `cycle` clears, `distill` compacts

- `lyx orch cycle` clears the context; a new verb `lyx orch distill` compacts it.
  `cycle_mode` in `orch.yaml` keeps choosing which of the two an automatic trigger runs.
- Both start the same way: lyx asks the orch to write a short note to a file lyx names, holding only what lyx cannot read elsewhere:
  - what the orch is doing right now, including any fork or edit half done;
  - what is agreed with the operator as the next step, if anything;
  - questions waiting on the operator.
  The note's template is a multi-line stencil, `contracts/stencils/orch/orch-note.md`, one heading per item with one sentence on what belongs there.
  The one-line instruction lyx types names the template and the note's path; it replaces `scribe:handoff` in `orch-template-handoff.md` and `orch-template-handoff-soft.md`, and the soft trigger keeps its `DEFER` reply.
  Notes are written where handoffs are today, one timestamped file per cycle under `.lyx/orch/handoffs/`, all kept.
- After `/clear`, or once `/compact` has finished (`/compact`'s focus argument only shapes the summary, and compaction drops the loaded skills' content), the watcher types, in order: the skills' slash commands, then the role stencil, then an instruction to read the note and continue.
- The same sequence runs at `lyx orch start` and `--adopt`, minus the note when there is none.

## The ly-drive skill becomes a stencil

The `ly:ly-drive` skill exists only to start the drivers that run a loom through Shed with repeated `lyx shed step`.
Every other Claude process lyx spawns gets its instructions from a stencil, and the driver should too.

- Move ly-drive's content into a stencil under `contracts/stencils/` and have the launch prompt render it.
- Remove the `ly-drive` skill from the `ly` plugin.
- Carry `loom-bouncer-rounds`' edit to ly-drive (its D6, the `parent_notice` to the parent) into the stencil.

## Agents escalate to their parent, and act on the parent's answer

Seen repeatedly on 2026-10-05: a spawned agent (a Webster Master, a driver) stopped and asked the operator in its own pane instead of its parent, and an agent that did ask the parent then refused to act on the parent's answer and asked the operator again.
In `loom-bouncer-rounds` the Master asked the hub how to handle an environmental verify failure, got "cherry-pick 7ff4cfc1", and then asked the operator, citing that a message from another session cannot grant permission and that its stencil forbids it to run git.

Operator decision (2026-10-05): agents never stop to ask the operator; they ask their parent (`LYX_PARENT`), and the parent asks the operator only what it cannot settle.

- Every role stencil names its parent as the one to ask, by name, and says a question to the operator in the pane is never the way forward.
- Every role stencil says a message from its own parent (`LYX_PARENT`) is the operator's delegate and is acted on like an operator instruction, within the role's limits; Claude Code's default treats a cross-session message as an untrusted peer, and the stencil has to say otherwise.
- A parent's instruction that falls outside the role's limits (the Master may not run git) names who carries it out instead, so the parent never gives an instruction the recipient cannot follow; the orch stencil says which role can do what.

Settle in Discussion which spawned roles also get the scribe skills typed in at launch: every role that writes prose for the operator or into the repo.
