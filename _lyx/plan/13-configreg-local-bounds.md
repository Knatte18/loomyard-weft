# Card 13 — configreg local bounds

**Create:**
- `plan:internal/configreg#TestModules_EveryTemplateKeyClassified` -> `func TestModules_EveryTemplateKeyClassified(t *testing.T)`
- `plan:internal/configreg#TestModules_LocalLayerRefusals` -> `func TestModules_LocalLayerRefusals(t *testing.T)`
- `pattern/PATTERN-config-layers.md`

**Edit:**
- `internal/configreg#Module`
- `internal/configreg#Modules`
- `internal/configreg#TestModules_SeedOnlyAndHubWideFlags`
- `PATTERN.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/configengine#Spec`
- `internal/configengine#LocalKeys`
- `internal/configengine#LocalDenied`
- `internal/configengine#Layers`
- `internal/batcher#ConfigSpec`
- `internal/boardengine#ConfigSpec`
- `internal/burlerengine#ConfigSpec`
- `internal/darnengine#ConfigSpec`
- `internal/fabricengine#ConfigSpec`
- `internal/gateslot#ConfigSpec`
- `internal/landingshed#ConfigSpec`
- `internal/loggerconfig#ConfigSpec`
- `internal/loomengine#ConfigSpec`
- `internal/modelspec#ConfigSpec`
- `internal/orchengine#ConfigSpec`
- `internal/reedengine#ConfigSpec`
- `internal/shuttleengine#ConfigSpec`
- `internal/websterengine#ConfigSpec`

**Intent:**
Give the registry D4's two lists and the pin that ends enumeration by hand.
`configreg.Module` gains three fields: `Spec func() configengine.Spec`, the module's own `ConfigSpec`; `LocalKeys []string`, the keys of a hub-wide module a local override may set; and `LocalDenied []string`, the dotted key paths of a per-worktree module a local override may not set.
`Modules()` wires each entry's `Spec` to its package's `ConfigSpec` and derives the two lists from it through `configengine.LocalKeys` (hub-wide entries) and `configengine.LocalDenied` (per-worktree entries), so the engine's `Categories` are the one source and the registry never restates a key.

`TestModules_EveryTemplateKeyClassified` walks every module's template units (`yamlengine.LeafPaths` with the module's open maps and `WholeEntries`) and asserts each is covered by a `Categories` entry, exactly or through an ancestor, and that every `Categories` entry names a path the template holds; a key added to a template later fails it until it is classified.
It also pins the derived lists against the record: darn's `LocalKeys` are `verify_attempts`, `writer` and `writer_timeout_min`, landing's `conflict`, `conflict_timeout_min`, `describe` and `describe_timeout_min`, board, fabric and gate have none; reed's `LocalDenied` are `debug_log`, `mouse`, `shell`, `tmux` and `tmux_conf`, shuttle's `claude`, the three `claude_deny_*` keys and `run_dir`, loom's `discussion_fan`, `discussion_producer`, `parent_review_wait_min` and `plan_fan`, burler's `fans` and `lenses`, and every other per-worktree module has none.
`TestModules_SeedOnlyAndHubWideFlags` gains the agreement check that each entry's `Spec()` carries the entry's own name, template text, open maps and hub-wide flag.
`TestModules_LocalLayerRefusals` is the acceptance table for the local-layer keys no loader card's wiring row pins, each key's verdict pinned once across the plan: over a temp board dir and anchor, a local file holding each of `darn.review_max_bounces`, `landing.require_pr_to_base`, `reed.mouse`, `loom.discussion_producer` and `loom.plan_fan` makes `configengine.Layers(dirs, m.Spec())` fail with `ErrInvalid` naming the local file and the key, while `shuttle.claude_prompt_cache_ttl_roles.<role>` passes.
The keys the loader cards pin through the real load (cards 4, 5, 8, 9 and 10) are not repeated here.

`PATTERN.md` gains, after `PATTERN-config-strictness`, the entry line

```markdown
- `PATTERN-config-layers` — Resolving a module's config: only `configengine` reads or stacks layer files and forms their paths; a key meeting a category the background names is closed to the local layer through `configreg`'s `LocalKeys` and `LocalDenied`. — [background](pattern/PATTERN-config-layers.md)
```

and `pattern/PATTERN-config-layers.md` states the four categories (runs an executable; sources a file into a shared process; reaches across pairs, including a shared process's boot option or what lands on the parent branch; switches off or hollows a guard, including a review or an approval gate, a timeout whose expiry lets that gate pass, and the prose or lens set a reviewer is given), that the test asks what a key's value can do to a guard rather than what kind of key it is, that each module classifies every template key in its `ConfigSpec` and a new key takes the same test, that local override files are never disposable for `lyx-cleanup` or any other cleanup, and that the git-ignored per-worktree `.env` stays an untracked route for marker-carrying denied keys (R3), which the local layer must not widen.
The budget row for `internal/configreg` rises by two.

**ImpactSummary:** adds three registry fields wired from the engines' specs; no caller of `Modules()` changes behaviour, and the flags test gains an agreement check.
