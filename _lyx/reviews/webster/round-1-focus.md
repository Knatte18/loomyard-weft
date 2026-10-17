---
round: 1
focus:
  - "Card 8 (retire-row-maps): confirm no reference to the removed per-row `Env` maps, `RowModelKeys` or their wiring helpers is left anywhere in the diff's packages, including battencli and battenrecipe, and that the Delete group's assert-no-callers holds."
  - "Cards 6 and 7: check that the review entries' build-time refusals fire for a missing segment or unnamed setting, and that budgets keyed by segment match the loom and darn segment names in `internal/loomcli` with no new shared constant."
  - "Card 3 and 4: check that `Env.Segments` and `Env.Setting` are legal nil in envkit's `nilLegal`, and that `envkit.TemplateSetting` fills `FullEnv` consistently with the real wiring from card 5."
  - "Declared PATTERN change: check that `PATTERN-recipe-setting-lookup` lands in `PATTERN.md` or `pattern/` with the declared text."
  - "Commit messages in the review range: check any `Pattern-Departure:` trailer names a known entry that forbids the card's way."
  - "Test economy: check new tests in configreg, shedrecipe and loomrecipe for duplication of existing coverage, and that new behavior (setting lookup, row identity) has coverage."
---

Seed focus drawn from the plan's card index and PATTERN change; no prior round exists.
