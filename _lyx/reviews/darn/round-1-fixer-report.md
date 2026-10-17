## Changes

- F1: `internal/configcli/doc.go` names `reviewer_model`, `fixer_model`, each segment's `*_reviewer_model` and `*_fixer_model`, and the `*_reviewer_fan` and `*_fixer_fan` keys, with darn's two fan keys. Fixed as proposed.
- F2: `copyWebsterModelsOntoMerge` in `internal/loomengine/config.go` copies the renamed `webster_reviewer_model` or `webster_fixer_model` value onto Merge when the document holds it, so Merge follows the value the rename keeps. The reported line names the source key.
  The function doc and the `MigrateConfig` doc say so. A both-names row is added to `TestMigrateConfig_RetiredKeys`.
  Fixed differently: the review proposed skipping the copy or copying from the new key, and I took the second.
- F3: a row with `darn_reviewer_model: []` in the darn `TestLoadConfig` table, asserting the load succeeds with an empty list. Fixed as proposed.
- F4: one row in `TestPrepareRunSettings` with the retired `webster_review`, asserting the renamed key and the Merge copy.
  The model keys pass the run-settings guard, so the row asserts the migrated document instead of a refusal.

Tests green: `lyx gate test` on `loomengine`, `darnengine`, `configreg` and `configcli`.
Commits: one per finding, plus a follow-up for the `MigrateConfig` doc under F2.

## Disputed

- Self-check LOW, `runsettings_test.go` row for F4 (lenses fix-code-correctness and fix-proposal): the row embeds the template's line comments, so a template comment edit breaks it. Ground `premise false`: carrying the template's line comment onto a renamed and a copied key is behaviour that `TestMigrateConfig_RetiredKeys` also pins, and the table compares whole documents, so a change to it should fail here too.

## Deferred

None.

## Self-check

- fix-consistency, LOW: the `MigrateConfig` doc at `config.go:718` named only the retired Webster keys. Fixed as proposed.
- fix-code-correctness, LOW: F4 row embeds template comment text. Disputed, see above.
- fix-proposal, LOW: the same finding. Disputed, see above.
- fix-affected-set: no findings.
