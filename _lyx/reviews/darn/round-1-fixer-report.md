# Fixer-report, round 1

## Changes

- **F1** fixed as proposed (amend, not fold).
  `PATTERN-shed-recipe-registry` in `PATTERN.md` now excepts the `Implementer` table, and the `registry.go` file header and `registry` comment name the table instead of calling the registry the only map.
  The gate then found the new `registry.go` comment lines fixed-column wrapped; the `registry` comment and the file header now hold one sentence per line.
  The `PATTERN.md` wording is compact because the overview sits at its headroom line; three words in other entries were trimmed to stay under it.
- **F2** fixed as proposed.
  `TestImplementerEntry_ForwardsTheRestOfTheRowConfig` builds a row with `implementer_setting` plus a `gates` key and refuses a row with an unknown key.
- **F3** fixed as proposed.
  `TestConfigTemplate_ImplementerDefaultsToWebsterAndAcceptsALocalValue` pins the template default `webster` and that `ConfigSpec` declares `implementer` with category none.
  The local-layer acceptance is covered by that category, not by loading a local layer.
- **F4** fixed as proposed.
  `implementerEntry` refuses a non-string `implementer_setting` and an empty one, each with a way forward; rows `EmptyKey` and `NonStringKey` pin them.
  The review's `entries_implementer.go:441` does not exist; the refusal came from `configString` in `config.go`, reached through `readSetting`.
- **F5** fixed as proposed, by reverting commit 4edda33eb, which held only the `deliver_test.go` hunk.
  The change description's creel bullet is outside the working tree and is left to the caller.

## Self-check

- fix-code-correctness, MEDIUM: the F3 category assertion passed with the entry dropped, since none is the zero value. Fixed: it now also asserts the entry is declared.
- fix-code-correctness, LOW: the `GatesKeyBuilds` row does not prove forwarding by itself. Fixed differently: no change; the unknown-key row proves forwarding and the gates row is what the review asked for.
- fix-proposal, MEDIUM: the F1 `PATTERN.md` wording pushed the overview past its headroom line. Fixed: wording shortened, file is under the line.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal: no findings for F2, F3, F4 and F5.

## Disputed

None.

## Deferred

None.
