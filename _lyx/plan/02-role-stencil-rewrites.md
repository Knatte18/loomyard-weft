# Card 2 — role-stencil-rewrites

**Edit:**
- `contracts/stencils/orch/orch-template-role.md`

**Intent:** Rewrite the orch role stencil's prose to the operator's decisions D1 to D5, one sentence per line, naming no denied command and asking no agent to load a skill.
The `{{.command_index}}` and `{{.repository_rules}}` markers and the `## This repository` section card 1 added stay as they are.

`## Escalations from a child` (D1, D2, D3):
- The bullet "A hand-on of a review loop follows a fixed rule: accept unless a finding makes the design wrong, and hand a blocker that remains after round 3 to the operator." goes.
- In its place, a bullet: a review escalation whose stop report names an arbiter attempt whose `envelope.json`, in the attempt directory beside the verdict file the stop report names, carries verdict `hand-on` and an empty `cause` is the arbiter agent's own hand-on, and goes to a one-shot fork.
  The fork is handed the escalation brief's path and that `envelope.json`'s path, and follows the brief: it answers the envelope's `question` from recorded operator decisions only, as the brief defines them, records and resumes by the brief's steps, and otherwise records nothing and reports back, and you bring the question to the operator.
  The fork falls under the open-fork rules of `## Investigating a stop`: a repeat notice for the slug starts no second fork, and the slug is named in the orch note while its fork is open.
  The rule for deciding a stuck review is the arbiter's own, in `loom-template-arbiter.md`; the role restates none of it (`PATTERN-producer-pointer-rule`).
- After the bullets, one short paragraph (D2): a review escalation without the arbiter agent's own hand-on is a fault, handled under `## Investigating a stop` and never decided by you as a design call.
  It names the cases: cause `budget`; an arbiter refusal, which leaves no envelope; a `hand-on` carrying a `cause` (`timeout`, `died`, `missing`, `unparseable`, `precondition`); a job exit without an envelope; a failed re-launch after a `resume` verdict; and the arbiter switched off or out of spawns.
  When the fault is diagnosed and acted on and a design question still stands, you bring that question to the operator.
- The bullet "A design call the child cannot make: decide it and record it with `lyx loom decision add`." becomes: a design call the child cannot make, in an escalation that is not a review escalation: decide it and record it with `lyx loom decision add`; a review escalation never takes this bullet and follows the hand-on bullet or the fault paragraph.
- The peer bullet becomes (D3): each escalation naming a peer run is first classified as a hand-on or a fault; a fault is investigated as a stop.
  Two runs with related work never talk to each other; you align them: the peers' hand-on escalations go to one joint one-shot fork that reads both decision records under the same settle rule and bound, with each slug named in the orch note as an open fork.
  A peer's hand-on escalation that arrives while that fork is open waits for its report and the re-check, then goes to a fresh fork under the same rule that also reads the first fork's finished outcome.
- The `lyx loom circling continue` / `accept` bullet and the other mechanics bullets stay.

`## Investigating a stop` (D3): the paragraph "Work that spans runs stays with you: … review both PRs side by side at PR-Gate." becomes one sentence: deciding when held runs go on stays with you.
The comparing of plans and PRs, the reading of both Discussions and the pause after Plan-Write all go; the alignment agent compares related runs.

`## PR-Gate` (D4): "A small defect in that goal is fixed on the task branch;" becomes: a small defect in that goal is fixed by the run's own agents through the reject, since `lyx loom reject` sends the findings to the recipe's rework row (`PR-Rework` in loom, a fresh `Darn-Write` in darn); the sentence's tail, new hardening goes on the board as its own entry, stays.

`## After landing` (D5): the `landed` bullet loses its tail ", and, while the hub's landing `deploy` key is empty, run `update-plugins.sh` by hand", and the bullet "Only while the landing `deploy` key is empty, at every wake, … run `update-plugins.sh` when `origin/main` holds a commit `origin/prod` lacks." goes.

`## Where a finding goes` (D5): "the workaround itself is written down only in the repository's CLAUDE.md, with the issue number, and removed with the fix" becomes: the workaround is written down with the issue number in the repository's `ORCHESTRATOR.md` when only you apply it, and in its `CLAUDE.md` only when every agent needs it, and removed with the fix.

`## Scratch and notes` (D5): "durable rules in git" becomes: durable rules in git, with durable orch-only rules in the repository's `ORCHESTRATOR.md`.

After this card the role stencil's own text names no "round 3" rule, no `update-plugins.sh`, no `origin/prod`, and no instruction to read both Discussions, pause after Plan-Write to compare plans, or review PRs side by side.
The acceptance criterion on the rendered role file holds for that stencil text: a repository's `ORCHESTRATOR.md`, filled into `## This repository` by card 1, may name `update-plugins.sh`, as this repository's does under D7, and that section is the repository's text, not the role's.

No test pins this static prose (`PATTERN-test-economy`); card 1's marker tests and render tests cover the file's markers.

**ImpactSummary:** Prose-only change to one shipped stencil; its markers and section order are unchanged, so no render code or test moves.
