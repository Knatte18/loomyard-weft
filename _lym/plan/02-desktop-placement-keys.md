# Card 2 — desktop-placement-keys

**Edit:**
- `internal/desktophook#Config`
- `internal/desktophook#ConfigSpec`
- `internal/desktophook#LoadConfig`
- `internal/desktophook/template.yaml`
- `internal/desktophook_test#TestLoadConfig`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`

**Uses:**
- `internal/beamsetting#WindowPlacementModes`

**Intent:** Add the desktop module's three placement machine keys (D5, D7).

`template.yaml` gains:
- `window_placement: "off"`, the machine default mode: `off`, `same` or `next`;
- a `tiling:` mapping holding `min_tile_width: 500` and `min_tile_height: 600`, the smallest tile in logical pixels.
Each comment names the machine-layer set command, `lym config desktop --layer machine --set <key>=<value>` from the prime (Review criterion),
and says what the key decides.
`window_placement` decides the desktop a new terminal window lands on
(`same`: its parent orch's desktop, `next`: the desktops after it, `off`: where KWin opens it, recorded only),
and `beam.window_placement` overrides it per task.
The two tile sizes decide the columns (`floor(width / min_tile_width)`, at most 5) and rows (`floor(height / min_tile_height)`, at most 2) of a desktop's grid.

`Config` gains `WindowPlacement string` (`window_placement`) and a `Tiling` struct field (`tiling`) holding `MinTileWidth` and `MinTileHeight` ints.
`ConfigSpec` classifies `window_placement`, `tiling.min_tile_width` and `tiling.min_tile_height` as `configengine.CategoryMachine`,
the dotted form a nested template key takes in `Categories`.
`LoadConfig` refuses, through `configengine.MarkInvalid` and naming the key with `configengine.KeyName`,
a `window_placement` that is not one of `beamsetting.WindowPlacementModes()`, and a tile size below 1.
Each refusal names its way forward: `lym config desktop --layer machine --set <key>=<value>` from the prime.
The package comment in `window.go` stays true: it describes the window command only, and the placement keys are read by the placement child (card 11).

Tests, `tier0`, rows in existing tests:
- `TestLoadConfig` (desktophook): `TemplateDefaults` also asserts `off`, 500 and 600;
  new rows `MachineOverrideSetsPlacement` (`same`, 400, 300),
  `UnknownPlacementRefused` (`sideways`, naming `desktop.window_placement` and the set command)
  and `ZeroTileWidthRefused` (naming `desktop.tiling.min_tile_width` and the set command).
  The local layer's refusal of a machine key is `LocalOverrideRefused`'s, and the classification below pins that the new keys are machine keys.
- `TestModules_EveryTemplateKeyClassified`: `wantMachineKeys` for `desktop` becomes
  `buttons`, `tiling.min_tile_height`, `tiling.min_tile_width`, `window_command`, `window_placement` and `window_wait_ms`,
  in the sorted order `configengine.MachineKeys` gives.
Together they add under 0.1 seconds to one `tier0` run.

**ImpactSummary:** Three new machine keys in desktop.yaml with load-time checks; every existing desktop key and its callers read as before.
