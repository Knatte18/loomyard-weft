# Card 3 — plan-own-entry-rule

**Edit:**
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`

**Intent:** Carry D3 and D5's Plan-review part.

In `loom-template-plan.md`, add a sentence stating that no card changes this task's own board entry's body or scope, because the run's board lock refuses such a change; the landing's status write, a `run_settings` write, a card that writes another board entry and a card that changes the board module's code stay allowed.
Widen Step 1's post-Discussion route, the paragraph opening "A scope addition that arrives after the Discussion is a design call the parent records.", to also name a drop of a decided step that changes this task's own board entry's body or scope: the writer asks its parent to record the drop, the parent records it with `lyx loom decision add` as an `Added after Discussion` entry, the writer plans without the step only once the record carries that entry, and with no parent recorded it stops and reports.
State the bound in the paragraph: the widened route admits a drop of an own-entry step only, and no other decided step is dropped through it.

In `loom-rubric-plan-review.md`, add an `## Also flag` bullet naming as a finding a card that changes the task's own board entry's body or scope; a card writing another entry, the entry's status or its `run_settings`, or changing board-module code is not flagged by it.
Amend the `**Fidelity to the answer key.**` bullet so a Decision step no card carries is accepted only when the step changes this task's own board entry's body or scope and the decision record carries an `Added after Discussion` entry recording its drop; any other uncarried Decision stays a finding as before.
The Plan writer is told (Step 1) that the record is its only source of scope, so the rubric's writer/reviewer symmetry note stays consistent with the widened route.

Constraints on the new text are those Card 2 names: no bare cross-repository `.md`/`.go` path token, no ask-the-operator phrase, no edit-rule phrase, no denied command, and no new template marker.
Write one sentence per line.
No test is planned: a stencil's static prose is exempt under `PATTERN-test-economy`.

**ImpactSummary:** Two stencil files read at call time by Plan-Write and the Plan-Review perch; no Go code or test asserts the changed sentences.
