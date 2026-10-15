# Role language skills: config module and wiring

Both batches reported `done`, and no recovery or re-fork was needed.

## Batch 01-language-config-module (head `a800ea2bd`)

Cards 1–5 added the hub-wide `language` config module in `internal/languageconfig` (loader and shared skill-merge helper), registered it, gave the darn writer `scribe:testing` and the told language list, made the Webster master and recovery strand load the list after `roleSkills`, made the merge conflict session load it at Merge-In and Finalize, and wired `lyx loom` to load the module and tell the list to darn, Webster and landing.
The digest listed these deviations from the batch's own targets: `cmd/lyx/configstrictness_test.go`, `internal/languageconfig/config.go`, `internal/languageconfig/config_test.go`, `internal/languageconfig/template.go` and `internal/languageconfig/testmain_test.go`.
The fork's reply attributes the `configstrictness_test.go` change to adding `internal/languageconfig` to the pinned `LoadOrTemplate` set, and it also edited `cmd/lyx/testdata/test-budget.yaml`, which the digest did not list.

## Batch 06-webster-wiring (head `d678a3a68`)

Cards 6–9 made `lyx webster` hub wiring load the module for `run` and `recover-batch` (a broken `language.yaml` refuses with `config_invalid`), made the Burler engine load the told list on every seat of a profile marked `LanguageSkills`, accepted `language-skills` in the recipe and `lyx burler run` profiles, and marked the Webster, Merge and Darn review and fix rows `language-skills: true`.
The digest listed one deviation: `internal/loomrecipe/languageskills_test.go`, a file card 9 declares as a Create target.

## Scope guard

Informational only for both batches; no batch was stuck or failed on it.
The plan-level verify gate runs after this turn ends.
