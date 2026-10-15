# Card 4 — darn-own-entry-rule

**Edit:**
- `contracts/stencils/darn/darn-template-write.md`
- `contracts/stencils/loom/loom-rubric-darn-review.md`

**Intent:** Carry D4 and D5's darn-review part.

In `darn-template-write.md`, state that no step changes this task's own board entry's body or scope, because the run's board lock refuses such a change; the landing's status write, a `run_settings` write, a step that writes another board entry and a step that changes the board module's code stay allowed.
Add the route, beside the existing `## A scope correction from your parent` section: when the board entry or the decision record asks for such a step, the writer leaves it out from the start and asks its parent to record the drop with `lyx loom decision add`.
Until the record carries that `Added after Discussion` entry, the writer names the omission in the change description's design decisions as an unrecorded parent correction.
Once the entry exists, the omission is no deviation from the amended board entry.
The writer stops and reports only when it has no parent to ask.
State the bound: the route admits a drop of an own-entry step only, and no other step is left out through it.
Keep the existing sentence that the writer never runs `lyx loom decision add` itself.

In `loom-rubric-darn-review.md`, add an `## Also flag` bullet naming as a finding a darn step that changes the task's own board entry's body or scope; a step writing another entry, the entry's status or its `run_settings`, or changing board-module code is not flagged by it.
Such a step goes through `lyx board` into the records repository, so the committed diff never shows it: the bullet names the evidence the reviewer reads instead, a claim in the change description that the run changed this task's own entry's body or scope, which the rubric already reads as the writer's claim.
A decision-record step asking for such a change is evidence only of the request, which the next sentence governs.
State that a step left out through the darn writer's own-entry route is no scope finding once the record carries its `Added after Discussion` entry, and that before then the omission is an unrecorded parent correction, filed under the existing `**An unrecorded parent correction.**` bullet as a MEDIUM `decision` finding deferred to the parent.

Constraints on the new text are those Card 2 names: no bare cross-repository `.md`/`.go` path token, no ask-the-operator phrase, no edit-rule phrase, no denied command, and no new template marker.
Write one sentence per line.
No test is planned: a stencil's static prose is exempt under `PATTERN-test-economy`.

**ImpactSummary:** Two stencil files read at call time by the darn writer and the Darn-Review perch; no Go code or test asserts the changed sentences.
