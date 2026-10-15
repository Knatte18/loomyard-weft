# Card 1 — config-key

**Edit:**
- `internal/websterengine#Config`
- `internal/websterengine#ConfigSpec`
- `internal/websterengine/template.yaml`
- `internal/websterengine_test#TestLoadConfig`
- `internal/configcli#TestDispatchLayers`

**Uses:**
- `internal/configreg#TestModules_EveryTemplateKeyClassified`

**Intent:** Add webster's `kickstart_pack_file` key (D1), a bool defaulting to `true`, beside `kickstart_pack`.

- `Config` gains the field `KickstartPackFile bool` with tag `yaml:"kickstart_pack_file"`, right after `KickstartPack`.
  Its comment says `true` renders the file layout (the pack in `<batch>-pack.md`, the prompt ending with the `## Start here` read line) and `false` the inline layout (the pack inside the fork prompt, no pack file), and that it has an effect only when a pack is built.
- `ConfigSpec` classifies `"kickstart_pack_file"` as `configengine.CategoryNone`, like `kickstart_pack`; `TestModules_EveryTemplateKeyClassified` in `internal/configreg` already fails a template key no category covers, so it pins this entry with no edit; the plan's `## verify:` runs `./internal/configreg/...` for it.
- `template.yaml` gains, after the `kickstart_pack` line, `kickstart_pack_file: true` with a trailing comment saying it picks the pack's layout (its own file, or inline in the fork prompt), that every begin-batch reads it afresh, and that it is flipped with `lyx config webster --set kickstart_pack_file=false`.
- No hub file is edited: `configengine` fills a template key the merged layers lack, so a layer without the key resolves `true`.

Tests:

- `TestLoadConfig`: `templateDefaults` and `absentKnobs` gain `KickstartPackFile: true`, so every row whose body omits the key, such as "kickstart_pack absent loads the template default", pins the template default (acceptance: a layer without the key resolves `true`).
  The "overrides round-trip" body gains `kickstart_pack_file: false` and its `want` `KickstartPackFile: false`.
  A new row "a kickstart_pack_file that is not a boolean is invalid" with `kickstart_pack_file: maybe` appended to the knob body expects `wantErrKey: "webster config"`, the `unmarshal webster config` error `LoadConfig` marks invalid.
- `TestDispatchLayers` gains one row on module `webster`, with `seedBoard` set to `configreg.Template("webster")`'s text (fetched beside `loomTemplate` at the top):
  "worktree layer refuses a kickstart_pack_file that is not a boolean and never syncs" sets `kickstart_pack_file=maybe`, expects `wantCode: 1`, `wantErr: "webster config"` and no worktree `webster.yaml`.
  It pins that `--set` refuses a non-bool value because webster's loader rejects it.
  That `--set kickstart_pack_file=false` is accepted rests on two existing pins: the "overrides round-trip" row above shows the loader takes `false`, and the row "set into an absent worktree file writes only that key" pins the generic `--set` write path.

**ImpactSummary:** One new field and one template key in webster's config; no reader consumes the field until card 3, and existing config files resolve the default unchanged.

**Commit:** 1: webster config key kickstart_pack_file
