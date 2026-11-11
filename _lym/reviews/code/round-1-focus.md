---
round: 1
focus:
  - "internal/loomengine/raddletable.go, interactive branch of raddleModeRules: does the rendered text still ask for the dependencies column by part slug, and does it now ask for recipe and profile columns, as the board entry's Idea says?"
  - "Same branch: does the text keep the prose to reasons and set-aside alternatives, as the Idea says, without contradicting the later instruction to revise the recipes and profiles with the operator?"
  - "Other raddle prompt sources and docs (stencils, specs, PATTERN.md) that describe the interactive table: does any still describe a table with dependencies only, so the diff leaves them inconsistent with the new columns?"
  - "The raddleModeRules doc comment: does it match the rendered interactive text, and does it follow the repository's comment conventions?"
  - "Tests that pin the interactive raddle text: does the diff change or add coverage, and does any existing assertion on the old table wording now fail or go stale?"
---

The range holds one Go file with prompt text.
The change description may not exist; check any claim it makes against the diff.
