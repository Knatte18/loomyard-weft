```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the run-profiles-ladder diff. I found no defects.

**List (function or branch, case run, result)**
- `decode`, empty or comments-only document: `Decode` returns EOF, so the result is empty profiles and defaults. Correct.
- `decode`, `null` document: `raw` is nil, so the range does nothing. Correct.
- `decode`, empty profile name: refused. Correct.
- `decode`, top-level `defaults`: skipped in the profile loop and parsed after it, so profile order cannot matter. A profile named `defaults` cannot exist as a profile. This matches the decision record's "cannot be named defaults". Correct.
- `decodeProfile`, old-shape profile (`light: {loom: {plan_max_bounces: 1}}`): the module value `1` is not a mapping, so it is refused with the profile, recipe and module named, plus a way forward. Correct.
- `decodeProfile`, `light: {}` (a recipe-less profile): accepted as an empty profile. Seed-Child then refuses it for any recipe via `Settings`, and `Default` refuses it at load. Correct.
- `decodeDefaults`:
  - A non-mapping value is refused.
  - A non-string recipe value is refused.
  - `loom: ""` is refused as an unknown profile.
  - A default naming a profile that lacks the recipe's block is refused.
  - The reserved `defaults` entry is never treated as a profile, because the profile loop skips it.
  
  Correct.
- `Settings`, profile without the recipe block (`docs` for `loom`): the error names the recipes the profile covers and the way forward. `prepareRunSettings` wraps it in `ErrInvalidRunSettings`, so Seed-Child routes to Stuck. Correct.
- `Default`, recipe not in the map: returns `""`, which `prepareRunSettings` treats as "settings alone". Correct.
- `prepareRunSettings`, task naming no profile and recipe with no default: goes to `configreg.PrepareRunSettings(settings.Settings)`. Correct.
- `prepareRunSettings`, `Load` returns `ErrInvalid` (old-shape file): wrapped in `ErrInvalidRunSettings`, so the row goes Stuck.
- `prepareRunSettings`, `Load` returns another error (unreadable file): returned as a hard error.
- `prepareRunSettings`, task naming no profile and an old-shape or unreadable file: now Stuck or hard error, where before the file was not read at all. This is the "Open risks" behaviour in the decision record, so I count it as intended.
- `seedChildProducer.writeRunSettings`, `ReadRunSettings` error: returns the reason before `Recipe` is set, which is fine. Otherwise `Recipe` is set to the resolved recipe: the board `type`, else `defaultChildRecipe`. The test pins `darn` and `loom`. Correct.
- `checkProfiles`, `slices.Sorted(maps.Keys(profile))`, `PrepareRunSettings` on each block: iterates in sorted order, so errors are deterministic, and names both the profile and the recipe. An unknown key inside a block is refused. Correct.
- `boardcli` `profiles`, `loaded.ByName[name]` and `loaded.Defaults`: `Defaults` is never nil, because `decode` initialises it, so JSON prints `{}` rather than null. Correct.
- Template (`internal/runprofile/template.yaml`): every perch's hard cap is rounds plus one. `speed` loom has Discussion 0, Plan 0, Code 1, and darn has Code 0. `docs` is darn-only with `darn.writer` set. `writer` is a real darn template key (`internal/darnengine/template.yaml:1`). Defaults are `loom: solid` and `darn: light`. Correct.
- `ConfigSpec`, `defaults` classified cross-pair: it comes from the template's top-level keys, so `defaults` is covered. Correct.

**Findings:** none.

**Tool calls (3 of 6)**
1. Read `LayerRunSettings` and the non-test callers of `runprofile.Load` and `Lookup`. My context lacked `LayerRunSettings`' body and the other call sites.
2. Grepped for the `hard_max_bounces` and `writer` key definitions and Seed-Child's use of `recipe`. This was to check that the template keys exist and where the recipe comes from.
3. Grepped for `hard_max_bounces` users and the `writer` template key. This confirmed `darn.writer` exists in `internal/darnengine/template.yaml`.
