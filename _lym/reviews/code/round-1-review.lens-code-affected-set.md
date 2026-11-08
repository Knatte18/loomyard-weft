```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. List of changes, with the set found and whether it was covered:

1. **`LayerFile` machine layer is no longer limited to hub-wide modules** (`layers.go:139`)
   - `readLayers`, `layerRank`, `layerPath` in `set.go`, `validateOverride`, and the `gateslot` callers call `LayerFile` or take the layer generically. The `HubWide` check was only in `LayerFile`. Covered.
   - Doc and comment sites updated: `configengine/doc.go`, `configcli/doc.go`, `configcli` comments, `hubwide.go`, `configreg/doc.go`, `PATTERN-config-layers.md`. Covered.
   - Left stale:
     - `pattern/PATTERN-lym-files.md:25` still says "A hub-wide module's machine override sits under the board dir's…". Not covered.
     - `docs/overview.md:513` still says "A hub-wide module's machine keys, such as gate's `slots` and `tier_parallel`, also take a machine override." Not covered. The diff edits `docs/overview.md` elsewhere (line 693) but not here.
2. **`selectedLayer` machine refusal** (`hubwide.go:117`)
   - A per-worktree module with no machine key gets the new `--layer local` refusal. A hub-wide module with none gets the existing refusal. The tests (`configcli_test.go`) cover both. Covered.
3. **New `sandbox_mode` key** (`config.go`, `template.yaml`)
   - Readers: `codexengine.go` `Prepare` and `OpenLaunch`, both updated.
   - Classified in `ConfigSpec` and in the `configreg` test. Covered.
   - The `loadConfigOrRefuse` way-forward text now names `machine`. Covered.
   - Docs: `codexengine/doc.go` and `docs/overview.md:693` updated. Covered.
4. **Launch line under `danger-full-access`**
   - `command_test.go`, `open_test.go` and `prepare_test.go` are updated. No other reader of the line was found by grep outside the docs/research benchmarks, which are historical. Covered.
5. **`machineKeyRefusal` removed** (`configreg_test.go`)
   - The helper was test-only and the diff removes all of its uses. Covered.
6. **Commit messages and departures**: the commits carry no `Pattern-Departure:` trailer and none is needed. The directive text I saw declares no PATTERN changes, so nothing was checked against a declared list. The `PATTERN-config-layers` edit lands.

Findings:

- **LOW, consistency, `pattern/PATTERN-lym-files.md:25`**: the line still says "A hub-wide module's machine override sits under…". A per-worktree module with a machine key (`codex`) now reads the same file, so the rule no longer describes where the file sits. Evidence: `PATTERN-lym-files.md:25`, against `layers.go:139`.
  - Fix: change it to "A module's machine override…".
- **LOW, consistency, `docs/overview.md:513`**: the line still says "A hub-wide module's machine keys, such as gate's `slots`… also take a machine override." Codex's `sandbox_mode` is now a per-worktree machine key, and `docs/overview.md:693` documents it.
  - Fix: reword to "A module's machine keys… (gate's `slots` and `tier_parallel`, codex's `sandbox_mode`)".

Tool calls (2 Bash calls, each a multi-command batch):
1. grep for machine-layer wording that still says hub-wide, plus `layers.go` 480–600 and `set.go` for the other `LayerMachine` consumers. The manifest lacked these.
2. grep for other `workspace-write`, `danger-full-access` and `sandbox_mode` readers. It found only historical benchmark docs.
3. Read `PATTERN-lym-files.md:25` and `docs/overview.md:511–515` to confirm the stale lines, plus a grep of `docs` and `shared-libs` for hub-wide machine wording. The manifest lacked these lines.
