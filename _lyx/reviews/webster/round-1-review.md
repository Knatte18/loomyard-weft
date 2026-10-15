---
verdict: APPROVED
---

Range: `product.parent` is `main`, no `_lyx/loom/rework` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`: eight prose files across the four cards.
`go test` over `contracts/stencils` passes, which covers the bare-path, ask-the-operator and edit-rule constraints the cards name.

## Rejected

- Card 3, plan template: the new paragraph sits directly after the existing scope-addition paragraph and leaves it intact.
  The drop route requires the record to carry an `Added after Discussion` entry before the writer plans without the step, so Step 1's "the record is the only source of scope" holds.
  Not a defect.
- Card 3, plan rubric: the `Fidelity to the answer key` amendment accepts an uncarried Decision step only when both conditions hold, an own-entry step and a recorded drop, and states that any other uncarried Decision stays a finding.
  Not a defect.
- Card 4: the writer's omission route and the reviewer's bullet agree.
  Before the record carries the entry, the writer names the omission as an unrecorded parent correction and the reviewer files it under the existing `An unrecorded parent correction` bullet as a MEDIUM `decision` finding deferred to the parent.
  Once the entry exists, neither side treats the omission as a deviation.
  The citation of that bullet is accurate.
  Not a defect.
- Cards 2 to 4: each new sentence carries "body or scope" and names the allowed writes (status write, `run_settings`, another entry, board-module code).
  No bare path token, ask-the-operator phrase, edit-rule phrase, denied command or new template marker appears.
  Not a defect.
- Card 1: the `CLAUDE.md` and `docs/overview.md` wording drops the "moves from the board entry" implication.
  The `## Documentation lifecycle` heading is untouched, and `PATTERN.md` and `pattern/` are unchanged.
  Not a defect.
- Test economy: the diff adds no tests, and a stencil's static prose is exempt.
