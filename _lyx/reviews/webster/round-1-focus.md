---
round: 1
focus:
  - "Cards 3 to 8 (per-package form rewrites): check that every invocable command in each package carries an `audience` annotation and that `Use`, `Short` and `Long` follow the one form, with no command left out, including shed verb bodies mounted under `loom`, `shed` and `batten`."
  - "Card 9 (board payload `Use`): check that each payload `Use` token equals its parser's declared key, that the coupling test pins this, and that `set-deps` addresses by `id` with refusals that name a way forward."
  - "Cards 10 to 12 (issue comments): check that comment fetch, count display in `intake list` and body append in `intake import` handle zero comments, filtered comments and fetch failure, and that each new behavior has coverage."
  - "Cards 13 and 14 (strand `lyx` path and role-file index): check that the index renders at every launch and reload site, that a missing or unreadable path turns into idle with the reason rather than a failed tick, and that only the recorded strand path is used as the index source."
  - "Card 2 (help module): check that `SetHelpCommand` followed by `InitDefaultHelpCmd` makes `lyx help` and `lyx help index --audience` work in a real execution, and that the registration oracle covers `SetHelpCommand`."
  - "Card 16 (form guard) and test economy: check that the tree walk is the single guard for the form, and that no per-package test duplicates its coverage or a covering existing test."
  - "Per card, confirm that the type-specific mechanical check for each group (Rename, Delete and the like) ran against every target of that group."
---
