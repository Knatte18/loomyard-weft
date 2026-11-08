## Goal

The shipped rule and docs name arras as the recipe for a tapestry-sized note

Takes: the landed arras recipe, perch mode, family filing and project release; main's text of `plugins/ly/skills/board-policy/SKILL.md` after commit c8e43ac4
Delivers: the board-policy skill and the orch role naming `arras` for a tapestry-sized note, `motif` alone for a split made later, and a project's release as one release; the orch role telling the orch that an operator decision on a running arras entry goes through `lym loom decision add --by operator`, never the entry body (D7's orch text); the board skill describing the arras run; the weaving glossary listing `arras` as the recipe and keeping `cartoon` reserved; `lym board` help naming `arras` among the recipes
Exports: nothing

## Scope

The part changes these packages:

- plugins/ly/skills/board-policy (about 30 changed lines)
- plugins/ly/skills/board (about 30 changed lines)
- contracts/orchrole (about 30 changed lines)
- docs/reference (about 20 changed lines)
- internal/boardcli (about 5 changed lines)

## Decisions

### D9 — The shipped rule names `arras`

Decision: the shipped instructions that pick a recipe for a board entry name `arras` as the recipe for a tapestry-sized note.
`motif` alone stays for a design decided now but split later, on code still changing.
`plugins/ly/skills/board-policy/SKILL.md` and `contracts/orchrole/orch-role.md` carry the rule and the release of a project as one release.
The rule this rewrites is the Motif-first rule that commit c8e43ac4 landed on main in `plugins/ly/skills/board-policy/SKILL.md`,
so the run takes main's text of that file before editing it.
`plugins/ly/skills/board/SKILL.md` describes the arras run.
`docs/reference/weaving-glossary.md` lists `arras` as the recipe and keeps `cartoon` reserved.

Rationale: the operator named `arras` the default for a tapestry-sized note, and the rule belongs to lym for every hub.

## Constraints

- `PATTERN-shed-recipe-registry`: arras declares `contract: loom`, and a verb that serves some loom recipes refuses on others by the recipe's rows, never its name (D4's mode, D7's refusal).
- `PATTERN-stencil-ownership`: the new rubric and the raddle stencil's mode text are read at call time through `stencilstore`.
- `PATTERN-sole-parsers`: D5's outcome checks live in `internal/raddleparser`, the proposal's sole parser, which takes the mode as an option.
- `PATTERN-refusal-way-forward`: the refusals of D4, D5, D7 and D8 name their way forward, and a test reaches each.
- `PATTERN-run-records`: post-Discussion decisions are written only by `lym loom decision add`.
- `PATTERN-dobby-runs`: released members start only through their parent's dobby, which D8 relies on and does not change.

## Auto-mode assumptions

- The perch's fixer is cold, not Raddle-Write re-entered (D2).
- A project release releases its draft members at once, and the project keeps the whole record (D8).
- Shape `beam` is refused in an arras run (D5).
- The family's project is the arras entry turned into a project (D6).

## Open risks

## Acceptance criteria

- Review: D9 — the board-policy skill, the board skill, the orch role and the weaving glossary name `arras` as the recipe for a tapestry-sized note and keep `motif` alone for a split made later.
