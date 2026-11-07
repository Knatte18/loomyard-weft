# A sampler that iterates the rendering of HEDDLE.md until it reads well

`heddle-md-readability` · Note · enhancement, heddle

## Seen (operator, 2026-11-07)

`HEDDLE.md` in the Bolt checkout, which `e25dcf34` renders, is hard to read.
It groups entries by target and gives each a From, Order and Targets list and its full text in a fenced block, so the reader scrolls through every shipped entry's whole text, and what lym sets cannot be told from what this repository sets without reading each entry's From line.
Empty targets each take a heading and a "No entry." line.

## Idea

A sampler run that iterates the rendering of `HEDDLE.md` with the operator: it renders this hub's real heddle store, shows the result, takes the operator's comments and renders again until the operator approves a layout, which is then built.

Points to try in it, none decided:

- Separate what lym ships, what this repository sets, and what is derived from the hub's config (such as `agents.yaml`), and show a hub override of a shipped entry plainly.
- An overview table first, one row per entry with its source, target and a one-line summary, before any full text.
- Leave out targets with no entry.
- For `claude-settings`, the merged result the worktree file gets, beside or instead of each entry's object.
