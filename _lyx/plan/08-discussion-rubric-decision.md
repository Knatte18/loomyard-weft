# Card 8 — discussion-rubric-decision

**Prosa:**
- `contracts/stencils/loom/loom-rubric-discussion-review.md`

**Intent:** D7: in the Discussion-Review segment, `decision` means a question only the run's parent can settle, since card 12 sends that class to the parent.
Rewrite the rubric's `## Finding class` section:

- `decision`, in this segment, is a question only the run's parent can settle: the board entry plainly outdated by landed work or contradicted by another entry or run, a design conflict with another run, a landed parent commit that contradicts a decision or makes one the writer cannot settle by citing or dropping it, or doubt about what the task is for.
- A choice the writer can make is never `decision`, and neither is a question an `Added after Discussion` entry or a board note titled `Decision for <slug>:` already answers; a writer's `## Open risks` line that such an answer settles is a `consistency` finding whose fix drops the line.
- Correct "Class never changes whether a finding is fixed": every class is fixed by the fixer except `decision`, which goes to the run's parent and never to the fixer.

The `design` sentence and the enumeration-gap sentence stay.
Static prose: no test reads it.
