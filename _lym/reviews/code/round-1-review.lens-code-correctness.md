```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens, running each added or changed branch on a concrete case. I traced these by reading the code and executed nothing.

**List (function or branch: case → result)**

| Function / branch | Case run | Result |
|---|---|---|
| `configengine.MovedKeyIn` / `refuseMovedKey` | empty doc; `discussion: {}`; `discussion: x`; `discussion:` null | correct per D4 (null counts as non-mapping) |
| `configengine.load` → `refuseMovedKeys` | old key in default, worktree, other layer | correct |
| `configengine.ValidateLocal` | `old_key: 2`; `area: {key: 1}` | correct |
| `configengine.Set` moved check | old key; `discussion={...}`; `discussion=x` | correct |
| `configengine.Unset` | two old keys in one call | correct (no validation, both removed) |
| `KeyName`, `KeyAtLine`, `NestPaths` | empty doc, nested line, shared area | correct |
| `yamlengine.MoveKeys` | null old key; both present; `code:` null plus old key; `code: scalar` plus old key | correct |
| `yamlengine.MoveKeys` | anchor and alias across moved keys | **wrong (F2)** |
| `configsync.moveOldKeys` | no table, parse error, three outcomes | correct |
| `reconcileBaseWorktreeFile`, `reconcileSparseFile` | moved-only change; file emptied by null drop | correct |
| `ReconcileHubWideAt`, sparse branches | present base; prime-seeded base; move error | correct |
| `ReconcileHubWideAt`, seed-only branch | present profiles file and base | correct |
| `ReconcileHubWideAt`, seed-only branch | absent file seeded from a flat prime copy | **wrong (F1)** |
| `moveBlocks` | non-mapping document; unparseable base | correct |
| `moveBlocks` | profile name holding a dot | **wrong (F3)** |
| `configreg.MoveSettings`, `MoveRunSettings` | empty object, null module, float whole number | correct |
| `configreg.refuseMovedKeys`, `PrepareRunSettings` | `discussion` object and scalar | correct |
| `LayerRunSettings` over nested areas | same area, different leaves | correct (`yamlengine.Merge` merges mappings recursively) |
| `CheckSettingKey`, `SettingLookup`, `valueAtPath` | moved key, path through a scalar, null leaf | correct |
| `RewriteRunSettings` | no old key; render failure; absent `board.json`; nil `ParentOf`/`PairExists` | correct |
| `boardCriticalSectionRollingBack` | `afterSave` or render failure; rollback failure | correct |
| `walkHub`, `reconcileHubWide`, `reconcileWorktree` | skipped write; commit failure; merge starting mid-walk (local files are in the snapshot) | correct |
| `carryMovedRunSettings`, `notifyMovedKeys` | error, zero lines | correct |
| `reviewbudget` `holdingLayer`, `valueAt`, `CheckResolved`, `CheckLocal` | nested path, non-map area | correct |
| `loomengine.LoadConfig`, `LoadTapestryConfig`, `ValidateLocalValues`, `FlagCodeAutoAccept` | nested decode, absent `code`, `auto_accept: false` | correct |
| `autonomySettingLookup`, `AutonomyForcedSetting` | forced and unforced keys | correct |
| `shedrecipe.settingRef`, operand pattern | `loom.split.reviewer_fan`; no dot | correct |
| `hubgeom.ReconcileBeforeLoad`; loom and shed pre-runs | outside a hub; non-git cwd | correct |
| `dobbyengine`, `boardengine.checkDesignPrefix` message changes | key rendering | correct |

Not traced: the remaining `loomcli`, `loomshed`, `shedadapters`, `landingshed`, `gateslot`, `hardenshed`, `driftengine` and `battencli` production hunks.

**Findings**

- **F1. LOW, `design`, `internal/configsync/configsync.go` (`ReconcileHubWideAt`, seed-only branch).**
  - **Summary:** A `profiles.yaml` seeded from the prime worktree's copy is never moved, and the operator's values are later lost.
  - **Case:** The hub file is absent and the prime copy holds `loom: {code_max_bounces: 5}`. `mergedBytes = existing`, but `moveBlocks` is gated on `m.RunSettingsBlocks && !fileAbsent`. The baseless follow then adds the template's nested `code.max_bounces` beside the flat key. The next walk sees both, drops the flat 5 and keeps the template value.
  - **Rule broken:** D5 and D6 require that a seed carries the values.
  - **Reach:** Narrow, since it needs a hub with no hub-wide `profiles.yaml` and a prime copy of it.
  - **Evidence:** the `case m.SeedOnly:` block in `internal/configsync/configsync.go`, read this fork.

- **F2. LOW, `design`, `internal/yamlengine/move.go` (`MoveKeys`).**
  - **Summary:** Moved keys are appended in sorted order of old name, so an alias can end up ahead of its anchor.
  - **Case:** `code_max_bounces: &n 2` followed by `code_hard_max_bounces: *n`. `code_hard_max_bounces` sorts and moves first, so the output has `hard_max_bounces: *n` before `max_bounces: &n 2`, which does not parse. The same happens when a moved key holds an anchor that an unmoved key aliases.
  - **Effect:** The next parse in the reconcile fails, so the walk and `lym config reconcile --apply` can never move that file. The load meanwhile refuses it and names that same command.
  - **Evidence:** `internal/yamlengine/move.go:79-131` (from the diff piece). I did not run it.

- **F3. NIT, `design`, `internal/configsync/configsync.go` (`moveBlocks`).**
  - **Summary:** The block prefix is built as `top + "." + second`, and `MoveKeys` splits the prefix on dots. A profile named with a dot, such as `v1.2`, is silently left flat, with no line and no error.
  - **Effect:** Seed-Child later refuses the old key.
  - **Evidence:** `moveBlocks`, and `MoveKeys`' `findValueNode(root, prefix)`. I saw no name rule in `internal/runprofile/runprofile.go` forbidding a dot (searched by grep only).

**Tool calls**
1. Read `LayerRunSettings`, the seed-only branch, `snapshotConfig` and `parentMappingOf`: unchanged code or elided diff context the manifest lacked.
2. Read `boardengine.New`, `stampDates`, the nil-check users and grepped profile name rules: callees of `RewriteRunSettings` not in context.
3. `git diff` of the `loomengine` config files: those pieces were not in the manifest (output was truncated and I did not open the saved file).
4. Read the loom and tapestry loaders, the autonomy lookup and the `shedrecipe`/`beamcli` hunks: production code not in the manifest.
5. Grepped board config wiring and read `configengine.Unset`: how `lym board` configures a board, and the unchanged `Unset` body, were not in context.
6. Read the board write tail, `yamlengine.Merge` and `settingRef`: callees one hop beyond the diff.
