# Per-machine environment variables, exported to every process lym starts for the hub

`machine-env-vars` · Task · feature, config, gate, drift, shuttle · running · Worktree-Create

Depends on: [`drift-skip-all-capture`](design-drift-skip-all-capture.md)

Imported from [issue #19](https://github.com/Knatte18/loomyard/issues/19).

## Problem

A hub needs environment variables whose values differ per machine, such as a path to a sibling checkout (`COMMONLIB_REPO` in the NORCE Models hub, which CommonLib-reading tests need).
Today the only place lym sets an environment variable for a run is `gate`'s `tiers.<n>.env`, and only in the shared hub file committed in _bolt.
The machine layer refuses it:

```
lym config gate --layer machine --set tiers.tier0.env.COMMONLIB_REPO=/path
key "tiers" is executable and cannot be set in a machine override
```

So a per-machine value has to be committed into the hub-wide config, where it is wrong on every other machine.

## Design (operator, 2026-11-03)

- Environment variables are configuration in their own right: a config module of their own, `lym config env`, a map from variable name to value.
- Layers: the hub layer, shared and committed in _bolt, and the machine layer, never tracked; the machine layer wins per variable.
  An env value is data, not an executable key, so the machine-override restriction on `tiers` does not apply to it.
- lym exports the variables to every process it starts for the hub: gate tiers and the shared build, Drift capture and compare, and agent sessions, so an agent running tests by hand sees the same values.
- Precedence, most specific first: a gate tier's own `tiers.<n>.env`, then the `env` module, then the inherited environment.

The issue also reported a `lym config drift` whole-map refusal, which landed as task `drift-config-whole-map-set`: `--set capture={...}` now replaces the map whole.
