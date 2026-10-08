# Card 12 — loom-fan-config

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#ConfigOpenMaps`
- `internal/loomengine#LoadConfig`
- `internal/loomengine#ResolveReview`
- `internal/loomengine/template.yaml`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/loomengine#TestResolveReview`
- `internal/configreg#TestLookup`
- `contracts/specs/refusal-spec.md`

**Intent:**
Decisions D1 and D2 in `loom.yaml`.
`Config` gains `DiscussionFan` (`discussion_fan`) and `PlanFan` (`plan_fan`), each a fan name from `burler.yaml`, template default empty (the segment runs solo), and `FanReview` (`fan_review`), a `ModelSpecList` with the grammar and last-entry-repeats rule of `review`, template default `sonnet[high]`.
`ConfigOpenMaps` adds `fan_review`.
`LoadConfig` validates `fan_review` like `review`, and resolves each non-empty fan key through `burlerengine.LoadConfig(baseDir)` and `burlerengine.ResolveFan`, which falls back to the embedded template (card 1);
a fan that does not resolve is refused naming the key, the unknown name and the fans that exist, with the way forward of setting the key to one of them or to empty to run the segment solo.
No `webster_fan` key exists.
`ResolveReview` resolves every `fan_review` entry at run start, and a segment whose fan key is set takes its reviewer list from `fan_review` while keeping its own fixer list;
a solo segment keeps its segment review list, else `review`.
The template's comments and the file header of `LoadConfig`'s file describe the three keys, the solo default and that `fan_review` is the forks' model too, since forks run on the reviewer session's model.
`refusal-spec.md`'s loom section adds `fan_review` to the model-spec-list row and gains a row for the unknown-fan refusal (`PATTERN-refusal-way-forward`).

Bound: the keys can only turn on a fan for Discussion-Review and Plan-Review; a value changed between rounds takes effect at the next round's wiring and can switch a segment between fanned and solo mid-generation, which changes the reviewer model but skips no gate.

`TestLoadConfig_Loads` covers the template defaults and set fans; `TestLoadConfig_Refuses` gains rows for an unknown `discussion_fan` and `plan_fan` naming the key and the existing fans, and an invalid `fan_review` entry.
`TestResolveReview` covers a fanned segment's reviewer from `fan_review` with its own fixer, and a solo segment unchanged.
`TestLookup` pins `fan_review` among loom's open maps.

**ImpactSummary:** Three new loom.yaml keys validated at load; ResolveReview changes the Discussion and Plan reviewer lists only when a fan key is set, and loomcli reads them in card 13.
