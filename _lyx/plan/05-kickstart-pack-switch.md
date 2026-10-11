# Card 5 — The kickstart_pack switch in webster.yaml

**Edit:**
- `internal/websterengine#Config`
- `internal/websterengine/template.yaml`
- `internal/websterengine_test#TestLoadConfig`

**Intent:**
Decision D3: `webster.yaml` gains a boolean `kickstart_pack` with a template default of `true`.
It turns the begin-batch kick-start pack on and off, so runs with and without the pack can be compared.
Card 7 reads it.

Add `KickstartPack bool` with yaml tag `kickstart_pack` to `Config`.
Its doc comment says that `true` renders the batch's resolved member glyphs into the fork prompt, and `false` renders the fork prompt as it was before the pack.
Add a `kickstart_pack: true` line to `internal/websterengine/template.yaml`.
Its trailing comment does three things:
- It names what the key turns on.
- It says each `begin-batch` reads it afresh.
- It gives the way to turn it off, `lyx config webster --set kickstart_pack=false`.

The key is per-worktree and read through `LoadConfig` like the other keys, which stays on `configengine.LoadOrTemplate` (`PATTERN-config-strictness`).
`configengine` already fills a template key a present file lacks, so an existing `webster.yaml` resolves the key to `true` with no `configengine` change.
A value that is not a YAML boolean fails the existing `yaml.Unmarshal` call, which `LoadConfig` already marks as `configengine.ErrInvalid`.
`TestConfigTemplate` already pins that every `Config` yaml tag has a template line.

Extend `TestLoadConfig`'s table:
- `templateDefaults` and `absentKnobs` carry `KickstartPack: true`.
- The overrides row sets `kickstart_pack: false` and reads it back.
- A new row has a hand-written file without the key and resolves it to `true`.
- A new row has `kickstart_pack: maybe` and fails marked `configengine.ErrInvalid`.

**ImpactSummary:** one new field on `Config` and one new template key; no reader acts on it until card 7, and existing `webster.yaml` files keep loading with the key filled to `true`.
