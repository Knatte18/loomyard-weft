# Survey Matt Pocock's skills repository for ideas that improve LoomYard

`pocock-skills-survey` · Task · research · landing · Finalize

## Question (operator, 2026-11-19)

Is there anything in Matt Pocock's skills repository that LoomYard could use: a skill, a prompt technique, a workflow, a guard or a tool that would improve lym's runs, its agents' prompts or its own code?

## Source

The local clone `/home/knatte/Code/pocock-skills` (origin `https://github.com/mattpocock/skills.git`), already up to date: `main` at 5863806 and the unmerged branch `origin/constrain-skill` at 31964fa.
Read it with `git -C` only; never write to it, never check out another branch there (read the branch with `git -C ... show` and `git -C ... ls-tree`).
`skills/deprecated` is out of scope.

## Exact cap

One pass over `skills/` (engineering, productivity, in-progress, misc), `docs/` and the `constrain-skill` branch's changes, by the writer alone, with no subagents, no tests run and no repeats.

## What to compare against

LoomYard's own skills under `plugins/`, its stencils under `contracts/stencils/`, the recipes (darn, tapestry, motif, raddle) and the rules in `pattern/`.
Skip anything LoomYard already does; say so in one line.

## Output

A report at `docs/research/survey/pocock-skills-2026-11-19/report.md`:
a table of candidates, each with what it does in his repository, where it would land in LoomYard, the expected gain, the rough cost, and a verdict (adopt, adapt, skip).
End with the three best candidates, each written so it can become a board note.
