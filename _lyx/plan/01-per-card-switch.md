# Card 1 — per-card-switch

**Edit:**
- `internal/websterengine#Config`
- `internal/websterengine#ConfigSpec`
- `internal/websterengine/template.yaml`
- `internal/websterengine_test#TestLoadConfig`
- `internal/configcli#TestDispatchLayers`

**Intent:**
Add the `webster.yaml` bool key `kickstart_pack_per_card`, default `true`, the switch that selects the per-card kick-start pack layout (D1).
Nothing reads it yet; card 3 makes begin-batch act on it.

- `Config` gains the field `KickstartPackPerCard bool` with the yaml tag `kickstart_pack_per_card`, placed after `KickstartPackFile`.
  Its doc comment says true renders the per-card layout, in which the fork fetches each card's pack with `lyx loom card-pack` at that card's start and no pack file is written;
  that it overrides `kickstart_pack_file`, which picks between the file and inline layouts only when this key is false;
  that, like the other pack keys, it has an effect only when a pack is built;
  that the per-card layout applies to a hub run, while a standalone run keeps the layout `kickstart_pack_file` picks because `lyx loom card-pack` cannot run there;
  and that every begin-batch reads the key afresh.
- `ConfigSpec` registers `kickstart_pack_per_card` as `configengine.CategoryNone`, as the other two pack keys are.
- `template.yaml` gains, after the `kickstart_pack_file` line, `kickstart_pack_per_card: true` with a trailing comment saying it picks the per-card layout and overrides `kickstart_pack_file`, that the per-card layout applies to a hub run while a standalone run keeps the layout `kickstart_pack_file` picks because `lyx loom card-pack` cannot run there, that every begin-batch reads it afresh, and that `lyx config webster --set kickstart_pack_per_card=false` restores the layout `kickstart_pack_file` picks.
  It is loaded through the same `LoadConfig` as the other two keys, so a hub layer lacking it resolves the template default `true`.
  `TestConfigTemplate` already fails when a `Config` field's yaml key is missing from the template, so it needs no change.
- `TestLoadConfig`: `templateDefaults` and `absentKnobs` each gain `KickstartPackPerCard: true`, which pins the template default and a hub layer lacking the key resolving it.
  The "overrides round-trip" row's body gains `kickstart_pack_per_card: false` and its want gains `KickstartPackPerCard: false`.
  A new row, "a kickstart_pack_per_card that is not a boolean is invalid", sets `kickstart_pack_per_card: maybe` and wants `wantErrKey: "webster config"`, as the `kickstart_pack_file` row does, since a yaml unmarshal error names no key.
- `TestDispatchLayers` gains a row, module `webster`, layer worktree, seeded with the webster template on the board, `setFlags: []string{"kickstart_pack_per_card=false"}`, exit 0, whose check reads the worktree `webster.yaml` back through `websterengine.LoadConfig` over the fixture's config dirs and asserts `KickstartPackPerCard` is false.
  It is the `lyx config webster --set kickstart_pack_per_card=false` round-trip acceptance criterion.

**ImpactSummary:** one new Config field and template key read by no code yet; the existing config tests gain rows, and no caller of LoadConfig changes.

**Commit:** 1: webster: kickstart_pack_per_card switch, default true
