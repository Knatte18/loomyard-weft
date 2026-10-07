# Card 15 — the orch resumes a child with `lyx loom resume`

**Edit:**
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils/claims_test.go`

**Uses:**
- `internal/loomcli/resume.go`

**Intent:**
The orch role stencil names `lyx loom resume` wherever it names `lyx loom start` for a halted child, after a circling decision and after a goto, and drops "message the driver by name to resume" as a separate step (D7, D8).

- Run loop item 2: a halted child is resumed with `lyx loom resume` in the task worktree, once its cause is known.
- Escalations from a child: after `lyx loom circling continue`/`accept`, run `lyx loom resume` in the task worktree to resume the run; a goto leaves the run paused, so run `lyx loom resume` in the task worktree to resume it.
- The closing line "A driver that is still alive you also message by name, telling it to resume." goes: the verb wakes a parked driver itself.
- One sentence says that when `lyx loom resume` refuses, its message names the way forward (`lyx batten run <slug>` to bring a dead driver back, or `lyx loom start`), and the orch takes it.

The stencil names no denied command and asks no agent to load a skill, as its header says.
`claims_test.go`'s orch role claims flip from `lyx loom start` to `lyx loom resume` for the circling and goto lines, gain a `must` for the halted-child line naming `lyx loom resume`, and gain `mustNot` claims for "resume it with `lyx loom start`" and for the message-the-driver step.
The bouncer escalation brief's own `lyx loom start` claim is a different stencil and stays.

**ImpactSummary:** One orch stencil and its claims; no Go behaviour changes.
