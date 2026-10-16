# Orch delegates board intake to a one-shot fork

`orch-intake-fork` · Task · enhancement, orch, board · running · Describe

## Seen

On 2026-10-16, right after a `/clear`, the orch ran `lyx board intake list` and `lyx board intake import` in its own session.
Both print an issue's full body, so one survey issue (#671) entered the orch context twice, about 14k tokens of a 71k session.
The orch needs only issue numbers and titles to know whether the inbox holds anything.

## Operator decision (2026-10-16)

Intake is delegated to a one-shot fork.
The orch itself only checks whether the inbox is empty, reading numbers and titles alone.
When it is not empty, a one-shot fork takes each open issue onto the board (new note, fold into an existing entry, or close as noise), gives it its module labels, and returns one line per issue.
A fork rather than a fresh agent, since the fork already knows the board policy and the operator's settled decisions.

## Idea

- `contracts/stencils/orch/orch-template-role.md`: the Where a finding goes and After landing sections say intake goes to a one-shot fork, and the orch checks the inbox only by number and title.
- `lyx board intake list` gets a compact mode (number and title only), or compact by default with the bodies behind a flag, so the orch check costs little.
- `lyx board intake import` prints a short result (slug, issue, labels) instead of the whole entry with its body.
