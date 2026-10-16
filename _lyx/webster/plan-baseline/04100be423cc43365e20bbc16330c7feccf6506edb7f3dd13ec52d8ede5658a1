# Card 8 — loom-per-round-fan-keys

**Create:**
- `plan:internal/loomengine#FanList` -> `type FanList []string`
- `plan:internal/loomengine#FanList.UnmarshalYAML` -> `func (l *FanList) UnmarshalYAML(node *yaml.Node) error`
- `plan:internal/loomengine#fanShapeError` -> `type fanShapeError struct`
- `plan:internal/loomengine#fanShapeError.Error` -> `func (e *fanShapeError) Error() string`
- `plan:internal/loomengine#retireFanReview` -> `func retireFanReview(document []byte) ([]byte, []string, error)`
- `plan:internal/loomengine#SegmentKeys` -> `type SegmentKeys struct`

**Delete:**
- `internal/loomengine#fannedReview`

**Edit:**
- `internal/loomengine/template.yaml`
- `internal/loomengine#Config`
- `internal/loomengine#ConfigOpenMaps`
- `internal/loomengine#ConfigSpec`
- `internal/loomengine#validateFanKeys`
- `internal/loomengine#MigrateConfig`
- `internal/loomengine#LoadConfig`
- `internal/loomengine#ReviewSettings`
- `internal/loomengine#ResolveReview`
- `internal/loomengine#resolveSegmentModels` -> `func resolveSegmentModels(segment string, reviewSpecs, fixSpecs ModelSpecList, runWide burlerengine.RoundModels, reg modelspec.Registry) (burlerengine.RoundModels, error)`
- `internal/loomengine#templateConfig`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/loomengine#TestMigrateConfig_RetiresSkipMergeReview`
- `internal/loomengine#TestResolveReview`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `internal/configreg#TestLookup`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configcli/doc.go`
- `docs/overview.md`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/shedrecipe#RowModelKeys`

**Intent:**
Make each perch's fan key and fixer fan key a per-round list in `loom.yaml`, in the grammar of the per-segment model keys, and retire `fan_review` (D2, D3).

`FanList` decodes one fan name or a block sequence of names, any other shape a `fanShapeError` that `LoadConfig` names by key as it names a `modelSpecShapeError`.
`fanShapeError.Error()` reads `value at line %d is neither a fan name nor a list of fan names; set the key to a fan name, or to a list of fan names, with "" for a solo or single round`, so the key-naming wrap carries a way forward as it does for `modelSpecShapeError`.
`Config`'s `DiscussionFan` and `PlanFan` become `FanList`, and it gains `DiscussionFixFan` and `PlanFixFan` (`discussion_fix_fan`, `plan_fix_fan`), whose empty entry is the single fixer and whose fan name is the twill fixer with that fan's lenses.
The template gives all four `""`, so a hub that sets nothing runs solo review and the single fixer in every round, and each comment states the scalar-or-list grammar, the last entry serving every later round and what an empty entry means; `fan_review` leaves the template.
`ConfigSpec` makes the two new keys guards like the fan keys, so all four are refused in the local layer and in a task's `run_settings`, and declares `fan_review` in `RetiredKeys` with `CategoryNone`, so every load drops it from the stacked document and a layer still holding it loads.
`ConfigOpenMaps` drops `fan_review` and adds the four keys, so reconcile and `--set` carry a list whole.

`validateFanKeys` checks each of the four keys: an empty list (`[]`) is refused as `loom config key %q: the list is empty; write "" to run the round solo or with the single fixer`;
every non-empty entry is resolved through `burlerengine.ResolveFan` at load, which `lyx loom start` runs before round 1, and a refusal names the key, the 1-based entry and the fan: `loom config key %q entry %d: <ResolveFan error>; define the fan in burler.yaml, or set the entry to ""`.
`burler.yaml` is still read only when some entry is non-empty, and its read failure keeps today's wording widened to the four keys.
`MigrateConfig` runs `retireFanReview`, which removes a document's `fan_review` key with the one reconcile line `fan_review: removed; a fanned round's reviewer now runs on its segment's own reviewer list` and returns a document without the key unchanged, so a second reconcile changes nothing.

`ResolveReview` stops resolving `fan_review`: each segment's reviewer list is its own `*_review` list when set, else `review`, fanned or not, and the forks inherit that model; `resolveSegmentModels` loses its fan parameter and `fannedReview` is deleted.
`ReviewSettings` gains `Discussion` and `Plan` `SegmentKeys`, the loom key each half's model came from: `discussion_review` when set, else `review`, and `discussion_fix` when set, else `fix`, and likewise for Plan.
`loomCLI.wire` fills `RowClusterFans` with each segment's `FanList` for its Bouncer and review rows, `RowFixFans` with `discussion_fix_fan` and `plan_fix_fan` for the two review rows, and `RowModelKeys` for the four Discussion and Plan Burler rows: the review row's `Review` key and the fix row's `Fix` key from `SegmentKeys`, each in module `loom`, and the fix row's `FixFan` key `discussion_fix_fan` or `plan_fix_fan`, so a fanned reviewer's engine refusal names its reviewer list's key and the two fix rows refuse an unrecorded round.
Webster, Merge and Darn rows get no fan list, fix fan list or `FixFan` key.

Tests:
`templateConfig`, the test helper's `Config` literal, drops `FanReview`, writes the two fan fields as `FanList{""}` and adds the two fixer fan fields at `FanList{""}`.
`TestLoadConfig_Loads` replaces its `fan_review` row with rows loading each of the four keys as a scalar and as a list, and adds a row whose repo-default layer still holds `fan_review` and loads with the key ignored.
`TestLoadConfig_Refuses` replaces the two `fan_review` rows with an empty-list row for each of the four keys, a row whose second list entry names an unknown fan, each asserting the key, the entry and the way forward, a row whose `discussion_fix_fan` is a mapping, and a row whose `plan_fan` list holds a mapping entry, which reaches `fanShapeError` through the list branch of `FanList.UnmarshalYAML`, each refused naming the key and the way forward;
its existing "unknown discussion_fan" and "unknown plan_fan" rows keep their key, fan name and `known fans: ` assertions but replace their `to empty to run the segment solo` assertion with `entry 1` and `set the entry to ""`, since a scalar value is entry 1 of the key and takes the new wording, while the "set discussion_fan with a malformed burler.yaml" row keeps its assertions, since the read-failure wording stays;
it adds a local-layer subtest table refusing each of the four keys, naming the local file and the key.
`TestMigrateConfig_RetiresSkipMergeReview` gains a row whose document holds `fan_review` and loses it with the reconcile line; its loop's second-pass check proves a second reconcile changes nothing.
`TestResolveReview` asserts a fanned segment's reviewer list is its own `*_review` list, else `review`, and checks `SegmentKeys` for set and unset segment keys; its bad-`fan_review` assertion is dropped.
`TestWire_ReviewKeysReachTheEnv` seeds per-round lists for the four keys and asserts `RowClusterFans`, `RowFixFans` and `RowModelKeys`, and that the fanned review rows' models come from the segment's own list.
`TestLookup` lists loom's new open maps.
`TestModules_EveryTemplateKeyClassified`'s `wantLocalDenied` entry for `loom` gains `discussion_fix_fan` and `plan_fix_fan`, in the sorted order the derived list has: each right after `discussion_fan` and `plan_fan`.

`refusal-spec.md`: the model-spec list row drops `fan_review`; the unknown-fan and unreadable-burler.yaml rows widen to the four keys and to a list entry, with the new way forward; a new row refuses the empty fan list; a new row, `config: fan key not a fan list`, refuses `discussion_fan`, `plan_fan`, `discussion_fix_fan` or `plan_fix_fan` holding a mapping or a list with a non-scalar entry, with `fanShapeError`'s way forward; the cluster-fan reviewer engine row names the segment's reviewer key in place of `fan_review`.
`internal/loomengine/config.go`'s file comment, `internal/configcli/doc.go`'s open-map example and `docs/overview.md`'s loom config paragraph name the four list keys and drop `fan_review`.

**ImpactSummary:** `Config`'s fan fields change type and `fan_review` goes, read only by `ResolveReview`, `validateFanKeys` and `loomCLI.wire`; a hub's reconcile drops `fan_review`, moving a fanned reviewer onto its segment's reviewer list, and every other loom key is unaffected.
