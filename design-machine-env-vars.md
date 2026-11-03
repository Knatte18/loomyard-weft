# Per-machine environment variables, exported to every process lym starts for the hub

`machine-env-vars` · Task · feature, config, gate, drift · draft

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

## Proposal

Support environment variables as configuration in their own right, possibly as a module of their own (e.g. `lym config env`), with a machine layer that is never tracked, so each machine sets its own values.
lym would export them to every process it starts for the hub: gate tiers, drift capture and compare, sessions.
An env value is data, not an executable key, so the machine-override restriction on `tiers` need not apply to it.

The issue also reported a `lym config drift` whole-map refusal, which landed as task `drift-config-whole-map-set`: `--set capture={...}` now replaces the map whole.
