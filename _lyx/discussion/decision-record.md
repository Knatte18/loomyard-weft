# Decision record — gate settings per machine

## Goal

`go_parallel` and `slots` take a machine-local override: a value set on one machine bounds every gate run of the hub on that machine and never reaches another machine through Bolt.
Today every gate key lives only in the hub file on Bolt (`<BoardDir>/_lyx/config/gate.yaml`), so a value tuned on one machine reaches every machine.
The lasting purpose is tuning the gate's CPU budget per machine, since the operator's machines differ in what bounds a run (Cortex XDR holds on OSL-1046, none on Neve), not only keeping a sweep's values off other machines.

## Scope

In:

- A new config layer, `machine`, for hub-wide modules, in `internal/configengine`: one never-tracked file per module under the board dir's `.lyx/config/`.
- A new key category, `machine`, that opens a key to the machine layer and keeps it closed to the worktree-local layer.
- Reclassifying gate's `slots` and `go_parallel` to that category.
- `lyx config` addressing the layer: `--layer machine` for `--print`, `--set` and editor writes, `--resolved` provenance, and the menu's layer marks.
- The gate load failure messages, in `gateslot`, `hubgeom.GateSlots` and `lyx gate test`, naming both files and both writes.
- Docs per the repo's same-commit rule, including `internal/gatecli/doc.go` and `internal/gateslot/doc.go`, which today say the limits come from the board's `gate.yaml` alone.

Out:

- `busy_quit_sec`: it stays hub-wide (`cross-pair`), since it is a hang detector not measured to differ per machine.
- A machine layer for per-worktree modules, and reclassifying any other module's key.
- Reconciling the machine file: no reconcile, `Ensure` included, reads or writes it (D5).
- Machine identity or ownership (task `machine-handover`); the layer is keyed by the checkout it lives in, not by a machine name.

## Decisions

### D1 — The machine layer is a never-tracked file beside Bolt's checkout, read by worktree loads only

The machine layer of a hub-wide module is `<BoardDir>/.lyx/config/<module>.machine.yaml`.
It is sparse and stacks between the hub file and the worktree-local file: template, default, machine, local.
It applies only when the spec is `HubWide` and both `Dirs.Board` and `Dirs.Anchor` are set.
A load with `Dirs{Board}` alone, which every hub-file check makes (`validateHubFile`, the hub-wide `CheckLoad` before and after a `--set`, and the hub-wide editor save), stacks no machine layer and so still checks the hub file alone.
`configengine` declares the path (beside `LocalFile`) and `LayerFile` returns it; no other package forms it.
An absent, empty or comment-only file adds nothing, as for the local layer.

Rationale: each machine has its own checkout of Bolt per hub, and its `.lyx/` is never tracked, so a file there is machine-local by construction and reaches no other machine.
The gate slot pool already lives at `<BoardDir>/.lyx/gate/`, so the slot budget's sharing domain is exactly this checkout: every worktree of the hub on this machine reads the same machine file through `fabricengine.ConfigDirs` and so agrees on the pool size.
Keeping hub-file checks machine-free stops a machine override from masking an invalid hub value, which would otherwise pass the check, reach Bolt and break every machine without an override.
The `.machine.yaml` name keeps the file apart from the editor staging copy at `<BoardDir>/.lyx/config/<module>.yaml`.

### D2 — A `machine` category opens a key to the machine layer only

`configengine` gains `CategoryMachine` ("machine"): a key whose value is a property of the machine, bounding a resource every worktree of the hub on that machine shares.
The machine layer admits only keys whose longest covering `Categories` entry is `CategoryMachine`, and a retired key by its recorded category under the same rule; a path no entry covers is refused, as the hub-wide local layer refuses one.
The local layer's rule is unchanged: it admits only `CategoryNone`, so a `machine` key stays closed to it.
`configengine` derives the open set as `MachineKeys(spec)`, beside `LocalKeys` and `LocalDenied`, and validates a machine document through one check shaped like `ValidateLocal`, with the same unit-path and shape rules; a failure is an `ErrInvalid` error naming the file and the key.
A per-worktree module that classifies a key `CategoryMachine` fails a `configreg` registration test, so the category is never declared where no layer can set it.

Rationale: the category model already answers "which layer may hold this key" per key; one more category keeps that the single classification, with no restated key list.

Bound: the category widens nothing for the worktree-local layer.
What it lets through is a value written to the machine file, which the D4 write guard confines to the prime, and a direct file write, which that guard does not stop (the default layer's existing bound).
The value still passes gate's loader at every acquire, so `slots` and `go_parallel` stay at least 1.

### D3 — Gate's `slots` and `go_parallel` are `machine` keys, and a gate load failure names both files

`gateslot.ConfigSpec` classifies `slots` and `go_parallel` as `CategoryMachine`; `busy_quit_sec` stays `CategoryCrossPair`.
`gateslot.LoadConfig` and `hubgeom.GateSlots` keep their signatures: the stack they already load through `fabricengine.ConfigDirs` now includes the machine file, read at every acquire as today, and `Lease.Env` puts the resolved `go_parallel` into `GOFLAGS` as today.
Every gate load failure message names the hub file and this machine's file and both ways forward: `lyx config gate` from the prime for the hub file, and for the machine file `lyx config gate --layer machine` or removing the file at its path.
This covers `withConfigWayForward`, the `GateSlots` wrapper, which today names the hub file alone, and the way forward `lyx gate test` adds after a failed limits read.
`lyx config gate --resolved` is named as the way to see which layer holds a value.

Rationale: the board entry asks for `go_parallel` and possibly `slots`; both bound how much of one machine's CPU the hub's gate runs take.
`slots` is machine-local safely because the pool's locks are machine-local, and `Pool.Limits` is a closure read at every acquire, so a lowered machine value takes effect on the next acquire while a holder of a higher slot finishes.
A value below 1 fails in `decodeConfig` after the merge with no file named, so the message must name both candidates.
Removing the file is named because `--set` cannot remove a refused key, and the editor path is refused without a terminal.

### D4 — `lyx config gate --layer machine` writes the machine file from the prime only

`configcli` accepts `machine` as a `--layer` value for `--print`, `--set`, an editor edit and the menu.
A machine write passes the same hub-write guard as a default-layer write (`hubWriteRefusal`): refused from any worktree that is not the hub's prime and from a session whose strand name carries a slug.
It is never committed and runs no fabric sync, and is checked after the write by the module's `CheckLoad` over the prime's worktree stack, which includes the machine layer per D1, and put back on a failed load as every `--set` is.
A key the machine layer refuses is refused before the write, naming the key and the layers that take it.
`--layer machine` on a module that is not hub-wide is refused with a way forward naming `--layer local`.
`--resolved` names `machine` as a unit's layer, and the menu marks a module whose machine file exists with `machine`.

Rationale: a machine value changes every gate run of the hub on this machine, so a pair agent must not set it, which is the default layer's guard already.
Bound: the guard stops a mistaken write by a pair or slugged session, not a direct file write, as for the default layer.

### D5 — The machine file is never reconciled

No reconcile, `hubreconcile.Ensure`, `lyx config reconcile` with or without `--layer default`, `lyx fabric reconcile` or clone, reads, rewrites or removes a machine file.
Every load drops retired keys from it, as from every layer, so a key retired with its category recorded still loads.
A key the machine layer refuses fails the gate load with the D3 message, and nothing else.

Rationale: the board-side reconcile commits every file it writes in `_board` and its rollback covers only `_lyx/config/`, so a `.lyx` file in that pass would break the commit or escape the rollback.
`Ensure` failures stop preflight, arm, orch start and clone, so a bad machine file reconciled there would block every run on the machine rather than only gate loads.
The machine file holds at most the two gate keys, so a stale key is rare, and the D3 message fixes it.

## Constraints

- `PATTERN-config-layers`: only `configengine` forms and reads the machine file's path and merges it; `configcli` goes through it.
- `PATTERN-lyxdirs-single-declarer`: the `.lyx` component comes through `configengine.ScratchDir`, never spelled.
- `PATTERN-durable-vs-ephemeral-state`: the machine file is never tracked and no run depends on it; with it absent, as on a fresh clone or an adopting machine, every load resolves from the hub file as today.
- `PATTERN-hub-wide-config`: the hub-wide module's tracked file stays at the board dir only; the machine file is its never-tracked counterpart there, never in a worktree.
- `PATTERN-config-strictness`: strictness stays on the base layer; an absent machine file adds nothing under `Load` and `LoadOrTemplate`.
- `PATTERN-told-geometry`: `configengine` derives the machine path from the told `Dirs.Board`.
- `PATTERN-refusal-way-forward`: each new refusal (key closed to the machine layer, `--layer machine` on a per-worktree module, a guarded machine write, a gate load failure from the machine file) names its way forward and a test reaches it.
- `PATTERN-config-layers` "Local files": the removal list is unchanged, since D5 removes no machine file.

## Auto-mode assumptions

- The board entry's "possibly `slots`" is taken as in scope (D3).
- The layer is named `machine` and the file `<module>.machine.yaml` (D1).

## Open risks

- Two lyx builds on one machine reading one hub, such as a `.dev-bin` build and the installed one, read the same machine file; an older build without the machine layer ignores it and runs on the hub value, and disagrees on `slots` with a newer one until it is updated.
- An operator who forgets a machine override sees a hub value change on Bolt not take effect on that machine; `--resolved` and the menu's `machine` mark are how they find it.
- Note `gate-language-neutral` plans to replace `go_parallel` with a general `parallel` in the hub-wide `language` module; that move must classify `parallel` as `machine` and carry the machine file's value, or the override is lost.

## Acceptance criteria

- With the hub file at `go_parallel: 4` and the machine file at `go_parallel: 8`, a worktree's gate load resolves 8 and a slot lease's `GOFLAGS` carries `-p=8`; with the machine file absent both carry 4.
- A load with `Board` alone ignores the machine file: with the machine file at `go_parallel: 8`, `lyx config gate --set go_parallel=0` from the prime is refused and the hub file is unchanged.
- A machine file under a board dir changes nothing for a standalone load (`Board` empty).
- The machine layer refuses `busy_quit_sec` and an unknown key, naming the file and the key; the local layer still refuses `go_parallel` and `slots`.
- A `CategoryMachine` key in a per-worktree module's spec fails the `configreg` registration test.
- `lyx config gate --layer machine --set go_parallel=8` from the prime writes `<BoardDir>/.lyx/config/gate.machine.yaml` sparsely, makes no commit on Bolt and runs no fabric sync.
- The same write from a pair worktree, or from a session whose strand name carries a slug, is refused with its way forward and leaves the file unchanged.
- `--set go_parallel=0` on the machine layer is refused by `CheckLoad` and the file is put back.
- `--layer machine` on a per-worktree module is refused naming `--layer local`.
- `lyx config gate --resolved` names `machine` for a key the machine file holds; the menu marks `gate` with `machine` when the file exists.
- A machine file at `go_parallel: 0` makes `lyx gate test` fail with a message naming both files, `--layer machine` and removing the file.
- `hubgeom.GateSlots` limits reflect a machine `slots` value written between two acquires.
- `hubreconcile.Ensure` and `lyx config reconcile --layer default --apply` leave an existing machine file byte for byte unchanged.

## PATTERN changes

- `PATTERN-config-layers` — changes: Resolving a module's config: only `configengine` reads, stacks and locates layer files, the hub-wide machine layer included.
- `PATTERN-hub-wide-config` — changes: A hub-level config module: `HubWide` in `configreg`, its tracked file on the board only, its machine override never-tracked beside it.

## Notes for the plan writer

- `pattern/PATTERN-config-layers.md` gains `machine` in its category list and a machine-layer section (open set, Anchor-and-Board rule, never reconciled); `pattern/PATTERN-hub-wide-config.md` gains the machine file; `pattern/PATTERN-durable-vs-ephemeral-state.md`'s background gains the machine file's path beside the `.local.yaml` line, with its index entry unchanged.
- `configcli`'s refusal text that lists `default, worktree or local` as the `--layer` values gains `machine`.
