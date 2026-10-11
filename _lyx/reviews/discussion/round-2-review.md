---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D4 (LocalDenied for reed, Bound line), R3
    summary: D4 leaves reed.mouse and reed.debug_log open to the local layer, yet both are applied globally to the hub's shared tmux server at its boot, so the bound that a local override never changes a fact shared across pairs is false
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D6, Acceptance criteria (migration walk)
    summary: D6's strip of keys equal to the layers below is not stated to treat an open map as one unit, and a leaf-wise strip inside an open map changes the resolved value because D1 carries an open map whole from the highest layer
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D4 Rationale
    summary: D4's rationale says the denied per-worktree keys today change only through a tracked file, but reed.shell, reed.tmux, reed.tmux_conf, shuttle.claude and shuttle.run_dir expand env-var markers at load, an existing untracked route
---

### [BLOCKING] D4's bound is false for reed.mouse and reed.debug_log

**Location:** `_lyx/discussion/decision-record.md` D4 (the `LocalDenied` list for reed and the `Bound:` line), and R3, which repeats that D4's lists bound what a pair agent gains to session tuning.

**Issue:** D4 denies reed `shell`, `tmux` and `tmux_conf` and leaves every other reed key open to the untracked local layer.
Its `Bound:` line says a local override can never change a key that "is a fact shared across pairs".
Two of reed's open keys are exactly that:

- `reed.mouse`: `internal/reedengine/lifecycle.go` `pinBootOptionsLocked` runs `set-option -g mouse <value>` on the boot that spawns the server, and `internal/reedengine/doc.go` ("Mouse boot pin") describes the server as the shared per-hub server.
  A pair whose `reed up` happens to boot the hub server sets the global mouse mode for every pair on it.
  The reed doc names `off` as non-neutral: wheel gestures then type arrow keys into the live agent of whichever pane is under the operator.
- `reed.debug_log`: the template comment says it "takes effect only on the boot that spawns the shared per-hub server", and `lifecycle.go` maps it to tmux's global `-v`/`-vv` flags.
  At `2`, tmux writes its protocol log, covering every pane's traffic in every pair, into the hub logs dir.

Both are changed from an untracked file that landing never shows (D4's own R3 point), so the record's claim that the denylist keeps cross-pair keys in tracked layers does not hold.
This is the same enumeration gap round 1 fixed for `shell`/`tmux`/`tmux_conf`, recurring in the method: D4 lists keys by the categories executable, sourced file, guard and run dir, and does not test each open key against its own fourth category, a server-wide or hub-wide effect.
I checked the other per-worktree templates (logger, orch, webster, loom, batcher, burler, models, the rest of shuttle and reed) and found no further key with a cross-pair effect: logger retention sweeps the worktree's own `LogsDir`, reed `watchdog` is read per worktree, reed `width`/`height` size one session.

**Fix:** add `mouse` and `debug_log` to reed's `LocalDenied`, extend D4's reason sentence to name "an option the hub's shared tmux server takes at boot", and add one of them to the `LocalDenied` acceptance criterion's examples.
State in D4 that the denylist was derived by checking every open key of each per-worktree module against the four categories, so the plan writer applies the same test to any key it touches.

### [MEDIUM] D6's strip is not stated to preserve resolution across open maps

**Location:** `_lyx/discussion/decision-record.md` D6 (the strip sentence) and the acceptance criterion "The migration walk seeds an absent repo default ... strips keys equal to the layers below".

**Issue:** D1 merges mappings key by key but carries an open map (`configreg.Module.OpenMaps`) whole from the highest layer that holds it.
D6 strips "every key whose value equals what the layers below resolve to", without saying at what depth the comparison runs inside an open map.
A leaf-wise strip changes the resolved config.
Concrete case: shuttle's open map `claude_prompt_cache_ttl_roles` has the template entries `driver: 1h` and `webster: 1h`.
A worktree file holding `{driver: 1h, webster: 1h, plan: 1h}` over a repo default holding `{driver: 1h, webster: 1h}` loses `driver` and `webster` to a leaf-wise strip, keeping `{plan: 1h}`.
D1 then carries `{plan: 1h}` whole, and `driver` and `webster` fall to the `5m` default.
batcher's `profiles` and loom's open maps have the same exposure.
The strip's purpose is to make files sparse without changing what any worktree resolves to; the record never states that invariant, and no acceptance criterion pins it.

**Fix:** state in D6 that an open map is compared and stripped as one unit, kept whole unless it equals the resolved map below, and that the strip never changes a worktree's resolved config.
Add an acceptance criterion: after the strip, every migrated worktree resolves every module to the same values as before, including a worktree whose open map differs from the default's in one entry.

### [LOW] D4's rationale premise overlooks env-var expansion

**Location:** `_lyx/discussion/decision-record.md` D4, Rationale ("the denied per-worktree keys today change only through a tracked file that landing flags").

**Issue:** the reed and shuttle templates set `tmux`, `shell`, `tmux_conf`, `claude` and `run_dir` through `${env:LYX_...}` markers, and `internal/yamlengine/resolve.go` expands those markers at load.
An environment variable is an existing untracked route to these keys, so "only through a tracked file" is false as written.
D4's decision still stands: an env var binds only the process that carries it, while a local file persists in the pair and reaches every later driver and spawn that loads the module.

**Fix:** reword the rationale to rest on persistence: today the denied keys persist in a pair only through a tracked file that landing flags, and an env var binds only the process that sets it.
