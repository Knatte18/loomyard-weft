```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, tracing the diff's changes to D1–D9 and the four "Added after Discussion" entries.

**Per change → decision** (grouped by purpose, from the file list and the diff pieces; not every hunk in every package was read)

| Change | Traces to |
|---|---|
| `KeyName`, per-package `keynames.go`, full names in messages | D3 (cards 1, 10, 11) |
| Template comments outside loom and tapestry (key sets unchanged with comments stripped) | D3 (card 12), D2 holds |
| `contracts/` stencils, specs, recipes, orch role; `pattern/`, `docs/` | D3 (cards 13, 14), recipes also D8 |
| `shellguard` literal `shuttle.` | added entry "six plan defects", item 6; D3 |
| claudeengine settings golden | forced by the notice text change; merge resolution |
| `MovedKeys`, `MovedKeyIn`, refusals in load, set and local layers | D4 |
| `NestPaths`, `merge-parent` budgets | D8 |
| `KeyAtLine` dotted paths, `reviewbudget` `holdingLayer`/`valueAt`, `SettingLookup` dotted paths | forced by D1/D8 |
| `loom.yaml`/`tapestry.yaml` nesting, decoders, readers, shipped profiles | D1, D8, D9 |
| Move tables, `run_settings`/recipe-reference refusals | D4 |
| `yamlengine.MoveKeys`, `configsync` moves, walk mail | D5 |
| `profiles.yaml` block move, `MoveSettings`, `RunSettingsBlocks` | D6 |
| `RewriteRunSettings` with rollback, `carryMovedRunSettings` | D7 |
| `ReconcileBeforeLoad` and its call sites, `reconcileWayForward` | D5 |
| `shedverbs` bounds, arming fixture stamp, prebootstrap push skips | added entry "bound the loop integration tests"; forced by D5 |
| `dobbyengine/tasksetting.go` conflict resolution | merge; keeps both sides (`keyName` plus the three-value return) |
| `PATTERN.md`, `PATTERN-config-layers` background | declared PATTERN changes |

I found no unrequested change.

**Per decision → changes**
- **D1, D2, D4–D9:** built by the rows above.
- **D3:** only partly built (finding 1).
- **F4 (input forms stay):** holds. `--set` and `run_settings` keep their forms, and no one-token form was added.
- **Six plan defects:**
  - (1) built: the circling test sets `EnableTraverseRunHooks` and drives `circling continue`.
  - (2) built: a `TestDispatchSet` row unsets two old keys.
  - (3) built: the "board with no old key is not written" row.
  - (4) built: hard cap 9 sits beside soft cap 7.
  - (5) code built: deferred `notifyMovedKeys`. Whether a test pins it belongs to the tests lens.
  - (6) built.
- **Loop-test bound:** built. The only two files holding `TestLoopIntegration_*` both carry bounds, and the hang cause is fixed by stamping the fixture hub.
- **Bare keys card 11 left:** largely not built (finding 1).

**Findings**

1. **BLOCKING, class `design`** (a recurring enumeration gap, filed once on the method)
   - **Location:** the packages named in the parent's added entry "Name in full the bare keys card 11 left".
   - **Summary:** the entry says "the run names every one in full before it lands", but most listed sites still print bare key names. This also leaves D3 unbuilt and violates the declared `PATTERN-config-key-names`. Most of these files are not in the diff at all.
   - **Method gap:** nothing enumerated message strings against the template keys. The fix needs that sweep, rendered through `configengine.KeyName` from named constants, not only the sites below.
   - **Evidence**, all paths under `internal/`:
     - `gatecli/memoryguard.go:220,288,299,304` (`memory_quit_mb`)
     - `gatecli/watchdog.go:156,168` (`busy_quit_sec`)
     - `gatecli/redundancy.go:49,54`
     - `gatecli/mutatecheck.go:29`
     - `gatecli/testmap.go:323,533`
     - `batcher/registry.go:30-39`
     - `batcher/config.go:129,133`
     - `batcher/estimate.go:155-192`
     - `battencli/overviewdobby.go:38,165` (`max_runs`)
     - `battencli/cli.go:513,520` (`driver_kind`, `shed.yaml's`)
     - `beamcli/land.go:276,290` (`the beam's pull_request`)
     - `beamcli/pr.go:326,422,459`
     - `webstercli/fixplan.go:186`, `webstercli/fixverify.go:221`, `webstercli/recoverbatch.go:401` (`recovery_timeout_min`)
     - `hibernatecli/cli.go:112` (`hibernate.yaml's resume_on_wake`)
     - `preflightshed/preflight.go:99` (`batcher.yaml (its active: key …)`)
     - `weaverengine/config.go:90,96,100` (`ResolveLadder`, "the ladder key of weaver.yaml")
     - `sheddriver/tackler.go:201` (`tackler_max_spawns`)
     - `shedarbiter/call.go:194` (`arbiter_max_spawns`)
     - `burlerengine/fix.go:396` ("set engine: in shuttle.yaml")
     - `loomshed/plancontext.go:76,141` (`rename_tools`, `gate.yaml refuse_raw`)
   - **Not confirmed:** my searches matched no bare name in `reportcli`. `weaverengine`'s `settingRefusal` already names `tapestry.implementer` in full, though as a literal rather than through `KeyName`.

**Tool calls**
1. `git diff --stat` file list: the manifest had no list of changed files.
2. The shellguard and root pieces, plus greps of the configcli, beamcli and loomcli pieces: these were unread and carry the added-entry defect fixes.
3. Grep of the listed packages for bare template keys: those files were unread and mostly outside the diff.
4. Grep of `sheddriver`, `weaverengine` and `reportcli`, a template diff, and the list of `TestLoopIntegration_*` files: not in the manifest. The template diff output was unusable.
5. Second grep for bare names in the remaining listed packages: call 3 did not cover their `.yaml`-style wording.
6. Comment-stripped comparison of the other modules' templates against the base: needed to check D2, and call 4's version failed.
