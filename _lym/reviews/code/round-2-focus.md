---
round: 2
focus:
  - "internal/boardengine/store.go: does every board write path, UpsertTasksBatch as well as UpsertTask and any other caller of validateWrite or checkOfferedRecipe, refuse recipe arras for a new entry and for an existing entry of another recipe, judging the prior entry from what was stored before the write? Check what the tests in internal/boardengine/store_test.go drive."
  - "Does any user-visible text still offer arras as a recipe a board task may take? Look at internal/battencli/wire.go, internal/shedrun/seed.go, internal/runprofile/template.yaml, internal/configsync/testdata/profiles-flat.yaml, internal/loomengine/template.yaml, docs/overview.md and docs/reference/weaving-glossary.md, and check that code still runs an existing arras run."
---
