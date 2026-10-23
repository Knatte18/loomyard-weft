# Card 5 — landing-fixer-config

**Edit:**
- `internal/landingshed#Config`
- `internal/landingshed#ConfigSpec`
- `internal/landingshed#LoadConfig`
- `internal/landingshed/template.yaml`
- `internal/landingshed#TestLoadConfig_TemplateDefaults`
- `internal/landingshed#TestLoadConfig_InvalidKeys`

**Intent:** Add the merge fixer's hub-wide keys to `landing.yaml` (D4, D10), since darn and loom share Finalize and landing config is hub-wide (`PATTERN-hub-wide-config`).
New template keys, each with a line comment in the template's style:
`fixer: opus[high]`, the model-spec of the merge fixer session;
`fixer_timeout_min: 60`, the minutes one fixer turn may run;
`merge_fix_attempts: 2`, the fixer turns one Merge-In row entry may spend;
`finalize_fix_attempts: 2`, the fixer turns one Finalize row entry may spend;
`finalize_fix_max_lines: 40`, the added plus deleted lines a fixer change may hold before it is over the cap.
`Config` gains `Fixer`, `FixerTimeoutMin`, `MergeFixAttempts`, `FinalizeFixAttempts` and `FinalizeFixMaxLines`.
`ConfigSpec` classifies `fixer` and `fixer_timeout_min` as `CategoryNone` and the two attempt caps and the line cap as `CategoryCrossPair`, since they bound what lands on the parent unreviewed.
`LoadConfig` checks `fixer`'s model-spec grammar as it checks `conflict` and `describe`, and refuses a `fixer_timeout_min`, `merge_fix_attempts`, `finalize_fix_attempts` or `finalize_fix_max_lines` below 1, naming the key and the way forward `lyx config landing --set <key>=<n>` from the prime.
The defaults test pins the five values, and the invalid-keys test gains a row per refusal.

**ImpactSummary:** Five additive landing.yaml keys and their load checks; no existing key or reader changes.
