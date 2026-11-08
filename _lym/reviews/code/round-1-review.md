---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: plugins/ly/skills/board-policy/SKILL.md:47-48
    summary: never change the recipe of a task raddle filed is unqualified, yet the operator may still name darn for any task
    fix: In plugins/ly/skills/board-policy/SKILL.md reword the last line of the new bullet (lines 47-48) so the ban binds the session only, for example "never change the recipe of a task raddle filed on your own; the operator may still name darn for it".
    origin: lens:code-affected-set
---

### [LOW] Unqualified "never change the recipe" collides with the operator-names-darn rule

**Location:** plugins/ly/skills/board-policy/SKILL.md:47-48
**Issue:** The Goal says raddle's shape choice stands and is not changed.
The new bullet states that as "never change the recipe of a task raddle filed", with no actor.
The next bullet (line 50) lets a task run darn when the operator names darn for it.
Read literally, the two disagree for a raddle-filed task the operator wants as darn.
The restriction should bind the board session's own judgment, as the Goal's intent is that the session does not override raddle.

## Rejected

- Hand-split ban at `plugins/ly/skills/board-policy/SKILL.md:84-85` (lens:code-affected-set, scope): the previous bullet let a session split any entry, a Sketch included, by hand on approval.
  That contradicts the Goal's rule that raddle splits the Sketch, and the Goal says to bring any contradicting shipped sentence in line, so the rewording is licensed.
- Tension with "never propose one note alone as a loom task" (`SKILL.md:41-43`): the new bullet covers a note weighed for tapestry, which is a different case from a small change, so the two do not conflict.
- Unchanged `contracts/orchrole/orch-role.md:44-49` and `plugins/ly/skills/board/SKILL.md:63-71`: both describe what a recipe is or does and name no recipe-choice rule, so leaving them unchanged is consistent with the Goal.
- Raddle stencil and proposal spec (`contracts/stencils/raddle/raddle-template-agent.md`, `contracts/specs/raddle-proposal-spec.md`): the Goal says raddle's shape choice is not changed, and the diff leaves them alone.
- Commit message claims: the message matches the diff, and no `_lym/landing/summary.md` claim is contradicted.
- Test economy and per-card checks: the diff changes one skill file, so no test or card check applies.
- Assistant report: no findings; every item it listed agrees with the rejections above.
