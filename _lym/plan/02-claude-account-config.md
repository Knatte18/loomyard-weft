# Card 2 — Accounts and pins as machine-layer keys of the claude module

**Extends:**
- `internal/shuttleengine/claudeengine#Config`
- `internal/shuttleengine/claudeengine/template.yaml`
- `internal/shuttleengine/claudeengine/doc.go`

**Edit:**
- `internal/shuttleengine/claudeengine#ConfigSpec`
- `internal/shuttleengine/claudeengine#LoadConfig`
- `internal/shuttleengine/claudeengine#TestLoadConfig`

**Create:**
- `plan:internal/shuttleengine/claudeengine#AccountConfig` -> `type AccountConfig struct`
- `plan:internal/shuttleengine/claudeengine#AccountPins` -> `type AccountPins struct`
- `plan:internal/shuttleengine/claudeengine#validateAccounts` -> `func validateAccounts(claude *Config, home string) error`
- `plan:internal/shuttleengine/claudeengine#refuseTrackedAccountKeys` -> `func refuseTrackedAccountKeys(dirs configengine.Dirs) error`
- `plan:internal/shuttleengine/claudeengine#canonicalConfigDir` -> `func canonicalConfigDir(dir string) string`
- `plan:internal/shuttleengine/claudeengine#AccountConfig.retired` -> `func (a AccountConfig) retired() bool`

**Uses:**
- `internal/configengine#Layers`
- `internal/configengine#LayerDoc`
- `internal/configengine#MarkInvalid`
- `internal/configengine#CategoryMachine`
- `internal/shuttleengine/claudeengine#claudeHomeDir`

**Intent:**
D1's keys join claude.yaml.
`Config` gains `Accounts []AccountConfig` (`accounts`) and `AccountPins AccountPins` (`account_pins`).
`AccountConfig` holds `Name` (`name`), `ConfigDir` (`config_dir`), `Weight int` (`weight`) and `Windows map[string]int` (`windows`, window name to the used percentage at or above which the account is passed over in that window).
`AccountPins` holds `Hub` (`hub`) and `Beams` (`beams`).
The template gains `accounts: []` and `account_pins: {hub: "", beams: ""}`, and `ConfigSpec` classifies `accounts` and `account_pins` `configengine.CategoryMachine`, so `lym config claude --layer machine` writes them and the local layer refuses them (`PATTERN-config-layers`; new keys, so no reinterpretation).

`LoadConfig` validates after the merge, before it returns, and every refusal is marked with `configengine.MarkInvalid`, so `Prepare` and `OpenLaunch` refuse the launch through `loadConfigOrRefuse` and a bad value never degrades to no accounts:
- `refuseTrackedAccountKeys` reads `configengine.Layers(dirs, ConfigSpec())` and refuses a non-empty `accounts`, `account_pins.hub` or `account_pins.beams` in the default or worktree layer, also when the machine layer holds the key.
  The error names the key, the file, and the way forward: remove the key from that file with `lym config claude --layer default` or `--layer worktree`, and set it with `lym config claude --layer machine`; a row asserts that text.
- `validateAccounts` refuses: a name not matching `[a-z0-9-]+` or repeated; a `config_dir` that is not absolute after a leading `~/` is expanded to home; two entries whose directories compare equal; a negative weight; a window name outside `five_hour`, `seven_day`; a threshold outside 1 to 100; a pin naming no listed account or a retired one (weight 0).
  Each refusal names the key and `lym config claude --layer machine`.
  It rewrites each `ConfigDir` to its expanded, cleaned form, so every later reader compares one spelling.
- `canonicalConfigDir` is D1's comparison form: `filepath.EvalSymlinks` of the cleaned path, and the cleaned path when that does not resolve.
  Cards 4, 6 and 7 compare directories only through it.

`TestLoadConfig` (tier0) gains rows over temporary hub and machine files, each reaching its branch after every earlier check passes: a valid machine-layer list loads with `~/x` expanded; a duplicate name; an upper-case name; a relative path; two entries naming one directory through a symlink; a negative weight; an unknown window; thresholds 0 and 101; a hub pin naming an unlisted account; a beams pin naming a retired one; a non-empty `accounts` in the hub file; and a `account_pins.hub` in the worktree file while the machine file holds it too.
Each refusal row asserts the error names the key and `lym config claude --layer machine`.
The doc gains the account keys' paragraph.

**ImpactSummary:** Two keys added to claude.yaml's struct, template and spec; `LoadConfig`'s callers (`Prepare`, `OpenLaunch`, `PromptCacheLifetime`, the module's `CheckLoad`) gain only the new refusals, which an empty default never triggers.

**Commit:** 2: claudeengine: accounts and pins in claude.yaml's machine layer
