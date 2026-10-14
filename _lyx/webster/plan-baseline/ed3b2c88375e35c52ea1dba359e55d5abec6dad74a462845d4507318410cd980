# Card 8 — orch-role-hibernate

**Edit:**
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils#wordingClaims`

**Intent:**
Tell the orch how to hibernate the machine (Decision 11); card 5 already puts the verbs in the command index its role file carries.
The role stencil's `## Acting without asking` section gains a short paragraph: when the operator asks to hibernate the machine, run `lyx hibernate request`, with `--after <slug>` when the operator names a task to land first or `--after-all` when they name the runs in flight, then end the turn, since lyx itself pauses the hub, confirms every session paused, hibernates the machine and lifts its pause on wake, and queues a notice when it is done or failed; `lyx hibernate status` shows the request and `lyx hibernate cancel` withdraws it.
The paragraph names no machine sleep command, which card 6 denies, and asks for no skill and no color.
The orch role row of `wordingClaims` gains the claim `must: "lyx hibernate request"`, `section: "## Acting without asking"`, with the why that a hibernate request from the operator goes through the daemon, never a session's own call.

**ImpactSummary:** the orch's role file gains one paragraph and its claims row one claim; no other stencil changes.

**Commit:** 8: orch: route an operator's hibernate request through lyx hibernate
