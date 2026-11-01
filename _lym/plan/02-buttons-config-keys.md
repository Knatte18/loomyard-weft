# Card 2 — buttons-config-keys

**Edit:**
- `internal/reedengine#Config`
- `internal/reedengine#ConfigSpec`
- `internal/reedengine#ConfigTemplate`
- `internal/reedengine/template_posix.yaml`
- `internal/reedengine/template_windows.yaml`
- `internal/reedengine_test#TestLoadConfig_TemplateDefaultsResolve`
- `internal/desktophook#Config`
- `internal/desktophook#ConfigSpec`
- `internal/desktophook/template.yaml`
- `internal/desktophook_test#TestLoadConfig`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`

**Extends:**
- `docs/overview.md`
- `internal/reedengine/doc.go`

**Uses:**
- `internal/reedengine#LoadConfig`
- `internal/desktophook#LoadConfig`
- `internal/desktophook_test#newDirsWithMachineConfig`

**Intent:**
Add the `buttons` key to `reed.yaml` (D2) and to `desktop.yaml` (D3), carried as an uninterpreted YAML node so no load ever fails on a button's content.

reed:
- `Config` gains `Buttons yaml.Node` with tag `yaml:"buttons"`; its doc comment says the node is the `buttons` mapping of groups `task`, `beam` and `prime`, decoded and validated only at render (card 3), never by `LoadConfig`.
- `ConfigSpec` classifies `"buttons"` as `configengine.CategoryExecutable`, since an entry runs a command: the hub base and the worktree file may hold it and the local layer refuses it (`PATTERN-config-layers`).
- Both templates gain the shipped default set of D8 in the same order, with a line comment on `buttons:` naming the groups, the entry keys (`label`, `run`, optional `confirm`, optional `window: agents`), the placeholders `{slug}`, `{worktree}`, `{beam}`, `{session}` and `{strand}`, the prime's fallback to `beam` while `prime` is empty, and that an edit reaches a live session at its next attach or boot:
  - `task`: `HALT` `lym tender pause {slug}`; `CANCEL` `lym batten cancel {slug}` with confirm `Cancel {slug}?`; `code` `lym ide spawn {slug}`; `term` `lym reed window {worktree} --title {slug}`; `agent` `cd {worktree} && lym shuttle open --role agent --model 'opus[high]'`.
  - `beam`: `code`, `term` and `agent` as above, then `distill` `lym orch distill` with confirm `Distill the orch?`, and `refresh` `lym orch refresh` with confirm `Refresh the orch?`.
  - `prime: []`.
  The `agent` run is written as a YAML double-quoted scalar so the single-quoted `'opus[high]'` survives, since `opus[high]` is a bash glob.
- `ConfigTemplate`'s doc comment, which lists the plain-literal keys, names the `buttons` block among them.

desktop:
- `Config` gains `Buttons yaml.Node` with tag `yaml:"buttons"`, the machine's additional buttons, same shape as reed's.
- `ConfigSpec` classifies `"buttons"` as `configengine.CategoryMachine`, and its doc comment drops "Both keys" for "Every key".
- `template.yaml` gains `buttons:` with `task: []`, `beam: []` and `prime: []` and a line comment saying a machine adds status-bar buttons here, appended after reed's of the same place, set in `<BoltDir>/.lym/config/desktop.machine.yaml` by hand or `lym config desktop --layer machine --edit`.
  Each group is one list unit under `yamlengine.LeafPaths`, so the machine layer's shape check admits a non-empty list against the template's empty one.
  `configengine.ValidateMachine` refuses a group written as a mapping, empty or not, which fails desktop's load (the record's added-after-Discussion correction): an empty mapping is one unit, `buttons.task`, refused as a template shape mismatch; a non-empty mapping is descended into, so its units, such as `buttons.task.label`, are refused as keys not in the desktop template, since the template's `task` is a list.

`docs/overview.md`'s `internal/desktophook/` tree line gains "and the machine's added status-bar buttons".
`internal/reedengine/doc.go` gains a sentence beside the segment-colors one: the `buttons` block of `reed.yaml` is carried as an uninterpreted node and decoded only at render.

Tests (all existing files keep their tier tags, `tier0`):
- `TestLoadConfig_TemplateDefaultsResolve` asserts the resolved `Buttons` node decodes to exactly the D8 set: the `task` group's five entries in order, each with its `label`, `run` and `confirm` (CANCEL's `lym batten cancel {slug}` with `Cancel {slug}?`, the `agent` entry's `run` exactly `cd {worktree} && lym shuttle open --role agent --model 'opus[high]'`), the `beam` group's `code`, `term`, `agent`, `distill` and `refresh` with the two orch confirms, and `prime` an empty list.
  A subtest `BothTemplatesShipTheSameButtons` reads `template_posix.yaml` and `template_windows.yaml` from the package directory, decodes each file's `buttons` node and asserts the two are equal to that set, so the Windows template is checked on a Linux gate; it reads only its own package directory.
  Its rows gain: a worktree file `buttons: null` fails with `configengine.ErrInvalid` naming `buttons` (the template fill's shape mismatch, `yamlengine.FillMissing`); a worktree file holding `buttons: {task: {label: x}}`, a group written as a mapping, loads with no error; and the existing `LocalExecutableKeyIsRefused` row's shape repeated for a local file `buttons: {task: []}`, refused with `ErrInvalid` naming the local file and `"buttons.task"`, the unit `ValidateLocal` refuses.
- `internal/desktophook_test#TestLoadConfig` compares `WindowCommand` and `WindowWaitMs` field by field, since `Config` holding a `yaml.Node` is no longer comparable with `!=`, and gains rows: a machine override with `buttons: {task: [{label: x, run: y}]}` loads and its `Buttons` node's `task` group holds that entry; a machine override with `buttons: {task: {}}` fails with `ErrInvalid` naming `buttons.task` and "different shape"; one with `buttons: {task: {label: x}}` fails with `ErrInvalid` naming `buttons.task.label` and "not in the desktop template"; a local override `buttons: {task: []}` fails with `ErrInvalid` naming `buttons.task`.
- `TestModules_EveryTemplateKeyClassified`: reed's `wantLocalDenied` gains `"buttons"` and desktop's `wantMachineKeys` becomes `{"buttons", "window_command", "window_wait_ms"}`.

**ImpactSummary:** two config structs gain a field no existing code reads, two specs gain one classification each, three templates gain a key; the desktophook test's struct comparison changes because `Config` stops being comparable, and no production code compares either `Config`.

**Commit:** 2: Add the buttons key to reed.yaml and desktop.yaml
