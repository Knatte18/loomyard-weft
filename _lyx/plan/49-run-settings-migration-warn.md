# Card 49 — run-settings-migration-warn

**Edit:**
- `internal/configreg#PrepareRunSettings`
- `internal/configreg#TestPrepareRunSettings`
- `internal/configreg#TestLayerRunSettings`
- `internal/configreg#TestProfilesCheckLoad`

**Uses:**
- `internal/testkit/logcapture#Capture`

**Intent:** `configreg.PrepareRunSettings` logs each line a module's `Migrate` returns at Warn, where it discards them today (D14).
The hub's `profiles.yaml` stays seed-only and nothing rewrites a board entry, so the profile check and `LayerRunSettings` migrate each profile and `run_settings` document at every read: a renamed key reads as its successor and a retired key is dropped, never refused.
Since nothing rewrites them, the Warn repeats at every read until the operator edits the document; each Warn names the module and the migration's line, which names the key, with the way forward to rename or remove the key in `profiles.yaml` or in the entry's `run_settings`.
`checkProfiles` and `LayerRunSettings` reach it through `PrepareRunSettings`, so both warn.

`TestPrepareRunSettings` pins a loom document holding `webster_max_bounces` and `merge_max_bounces` preparing with `code_max_bounces` set and no `merge_max_bounces`, and, read through `logcapture.Capture`, one Warn line per migration line naming the module, the key and the way forward; `TestProfilesCheckLoad` pins that a profile holding both keys validates; `TestLayerRunSettings` pins that a task's `run_settings` holding them layers over a profile with the first read as `code_max_bounces` and the second dropped.

`PrepareRunSettings`' doc comment names the Warn and its repetition.

**ImpactSummary:** One logging change in the run-settings preparation; every profile check and run-settings layering inherits it.
