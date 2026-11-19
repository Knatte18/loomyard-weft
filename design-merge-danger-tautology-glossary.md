# Merge danger in descriptions, tautological tests in review, and a shared vocabulary

`merge-danger-tautology-glossary` · Task · enhancement, landing, stencil, burler, heddle, docs · running · Code-Burler-Fix

Three small changes, mostly in stencils and review rubrics, bundled into one run.

## Part 1: merge danger in every change description

Source: `docs/research/survey/pocock-skills-2026-11-19/report.md` (survey of Matt Pocock's skills repository, 2026-11-19), candidate 1, which holds the design.

### Idea

The change description gets a `## Merge danger` section: **Door** `one-way` or `two-way` (one-way when a revert does not undo the effect: a format another binary reads, a config reinterpretation, deleted data, anything pushed outward) and **Blast radius** in one word.
The darn and Describe stencils ask for it, `lym loom validate-description` refuses a description without it, and a Code-Review rubric line checks the door against the diff.
## Part 2: tautological tests as a Code-Review finding

Source: `docs/research/survey/pocock-skills-2026-11-19/report.md` (survey of Matt Pocock's skills repository, 2026-11-19), candidate 3, which holds the design.

### Idea

The Code-Review rubric's Test economy list gains a bullet: a test whose expected value calls the code under test, a shared helper or the same formula is a finding naming the test and the independent value it should assert.
`scribe:testing` gains the matching rule. A mutation survivor on a line whose covering test recomputes its value is evidence for the finding.
## Part 3: shared vocabulary

### Seen

Agents and the operator drift apart on words: the orch had to learn by hand that "profil" means recipe, that a board entry is never a "kort" (cards are Webster's), and that a decision record is never a "logg".
Today the vocabulary is spread over the `nicknames` heddle entry, the orch's memory, package docs and skills.
`docs/reference/weaving-glossary.md` lists weaving words usable as names; it does not define what lym's terms mean.

The idea comes from Matt Pocock's `domain-modeling` skill (survey `docs/research/survey/pocock-skills-2026-11-19/report.md`):
a glossary, free of implementation detail, updated the moment a term is settled, and an agent that calls out a term used against it.
There it is a skill steering a live chat; here it becomes structure.

### Idea

- **One glossary file** in the repository: one line per lym term (entry, decision record, recipe, Sketch, run, ...), its short meaning and the words never to use for it.
  Lines stay short; explanations belong in docs.
- **Loaded only where needed**, never through `claude-prompt` or `agents-file`, which reach every agent:
  - the orch, which talks to the operator and writes the board, gets it through a heddle entry with target `orch-prompt`;
  - Discussion, which writes decision records and settles new terms, gets it included in its stencil.
    Discussion calls out a term in the entry that conflicts with the glossary, and a term it settles goes into the glossary.
  - Writers, fixers and reviewers get nothing.
- The glossary may also ship as a skill such as `ly:vocabulary`, loaded on demand by agents writing docs, stencils or skills.
  A skill alone is not enough: an agent does not know it is using a wrong word, so it would not load the skill when it needs it.
- No lint (operator, 2026-11-19): the drift seen so far was in chat with the operator, which a lint never reads, and banned words such as "card" and "log" are right in other senses, so a lint would cost more than it catches.
- The `nicknames` heddle entry stays for this hub's own nicknames; whether lym's general terms ship in the binary for every hub is for Discussion.
