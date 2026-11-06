# drift: config reconcile leaves the retired {skip_file} placeholder in capture and compare commands

`drift-retired-skip-file-kept` · Note · bug, drift, config, reconcile

Imported from [issue #70](https://github.com/Knatte18/loomyard/issues/70).

Seen: after drift stopped rendering {skip_file}, a hub whose default-layer drift.yaml held '-e NORCE_BASELINE_SKIP_FILE={skip_file}' in capture.command and compare.command blocked a task run at Drift-Recheck with 'the drift current capture command holds {skip_file}, which drift no longer renders'. 'lym config reconcile --layer default --apply' from the prime did not remove the placeholder.

Workaround: the prime orch rewrote both commands without the placeholder with 'lym config drift --layer default --set', then the blocked run was resumed with 'lym loom resume'.

Expected: reconcile (or the upgrade that retired the placeholder) migrates or strips {skip_file} from existing drift commands, so no run blocks on it; and the Drift-Recheck error names a way forward that works from the session that receives it (a beam orch is refused hub-wide config writes).

## From issue #69

Imported from [issue #69](https://github.com/Knatte18/loomyard/issues/69).

Seen: in the NORCE Models hub, drift.yaml (default layer) held capture.command and compare.command with `-e NORCE_BASELINE_SKIP_FILE={skip_file}`, written when drift rendered that placeholder. Drift no longer renders {skip_file}, so every drifting run blocked at Drift-Recheck. `lym config reconcile --layer default` reported no change for drift, so it did not migrate the placeholder away.

Workaround: removed the placeholder by hand from the prime with `lym config drift --layer default --set 'capture.command=...'` and the same for compare.command. A beam orch cannot do this, since a hub-wide config write is refused there, so the beam's members waited on the prime orch.

Expected: when a drift placeholder is retired, `lym config reconcile` (or drift's config load) strips or rewrites it in existing hub configs, or drift renders a retired placeholder as empty with a warning instead of blocking the run.