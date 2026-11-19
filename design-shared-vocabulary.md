# Shared vocabulary: one glossary, loaded only where needed, linted everywhere

`shared-vocabulary` · Note · enhancement, heddle, stencil, docs, lint

## Seen

Agents and the operator drift apart on words: the orch had to learn by hand that "profil" means recipe, that a board entry is never a "kort" (cards are Webster's), and that a decision record is never a "logg".
Today the vocabulary is spread over the `nicknames` heddle entry, the orch's memory, package docs and skills.
`docs/reference/weaving-glossary.md` lists weaving words usable as names; it does not define what lym's terms mean.

The idea comes from Matt Pocock's `domain-modeling` skill (survey `docs/research/survey/pocock-skills-2026-11-19/report.md`):
a glossary, free of implementation detail, updated the moment a term is settled, and an agent that calls out a term used against it.
There it is a skill steering a live chat; here it becomes structure.

## Idea

- **One glossary file** in the repository: one line per lym term (entry, decision record, recipe, Sketch, run, ...), its short meaning and the words never to use for it.
  Lines stay short; explanations belong in docs.
- **Loaded only where needed**, never through `claude-prompt` or `agents-file`, which reach every agent:
  - the orch, which talks to the operator and writes the board, gets it through a heddle entry with target `orch-prompt`;
  - Discussion, which writes decision records and settles new terms, gets it included in its stencil.
    Discussion calls out a term in the entry that conflicts with the glossary, and a term it settles goes into the glossary.
  - Writers, fixers and reviewers get nothing.
- **A lint** flags a banned word in stencils, skills and docs, so agents that never read the glossary are still held to it.
- The `nicknames` heddle entry stays for this hub's own nicknames; whether lym's general terms ship in the binary for every hub is for Discussion.
- The glossary may also ship as a skill such as `ly:vocabulary`, loaded on demand by agents writing docs, stencils or skills.
  A skill alone is not enough: an agent does not know it is using a wrong word, so it would not load the skill when it needs it.
  The orch and Discussion therefore get the glossary structurally, and the lint enforces it.
