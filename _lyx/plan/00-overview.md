---
format: 5
approved: false
language: go
---

# Plan: gate settings per machine

Gate's `slots` and `go_parallel` gain a machine-local override: a never-tracked `<BoardDir>/.lyx/config/gate.machine.yaml` that bounds every gate run of the hub on this machine and never reaches another machine through Bolt.
Card 1 adds the `machine` layer and `CategoryMachine` to `configengine`, card 2 reclassifies the gate keys and rewords every gate load failure to name both files, card 3 lets `lyx config --layer machine` print, set and edit the file from the prime, and card 4 pins that the hub reconcile leaves it alone.

## Card Index

1 — config-machine-layer — add the machine layer, `CategoryMachine`, `MachineFile`, `MachineKeys` and `ValidateMachine` to configengine, with the PATTERN docs
2 — gate-machine-keys — classify gate's `slots` and `go_parallel` as machine keys and name both files and both ways forward in every gate load failure
3 — config-cli-machine-layer — `lyx config --layer machine` for print, set, editor, menu marks and `--resolved`, guarded like the default layer
4 — ensure-leaves-machine-file — pin that `hubreconcile.Ensure` leaves a machine file byte for byte unchanged

## Shared Decisions

- The machine layer applies only to a hub-wide spec with a `machine` key, judged on the told `Dirs` with both `Board` and `Anchor` set; every hub-file check loads `Dirs{Board}` and so never stacks it.
- No reconcile reads or writes the machine file; nothing in `configsync` or `hubreconcile` changes.
- An absent hub file stays `ErrConfigAbsent` whether or not a machine file is present: D3's two-file message covers a value one of the two files holds, and a missing strict base has only the hub-file way forward, `lyx fabric reconcile`, whose clause already names what a session lyx refuses it from does.

## PATTERN changes

- `PATTERN-config-layers` — changes: Resolving a module's config: only `configengine` reads, stacks and locates layer files, the hub-wide machine layer included.
- `PATTERN-hub-wide-config` — changes: A hub-level config module: `HubWide` in `configreg`, its tracked file on the board only, its machine override never-tracked beside it.

## verify:

```
go build ./cmd/lyx
lyx gate test --tags integration ./internal/configengine ./internal/gateslot ./internal/hubgeom ./internal/gatecli ./internal/configreg ./internal/websterengine ./internal/planglyph ./internal/configcli ./internal/hubreconcile
```
