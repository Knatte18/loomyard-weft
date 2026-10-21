# Gate settings per machine: a machine config layer

`go_parallel` and `slots` can now be overridden per machine.
A value set on one machine bounds every gate run of the hub on that machine and never reaches Bolt or another machine.
Before, every gate key lived only in the hub file `<BoardDir>/_lyx/config/gate.yaml`, so a `go_parallel` sweep on one machine changed the other machines' runs too.
The machines differ in what bounds a run (endpoint security holds new binaries on one, none on the other), so the CPU budget needs tuning per machine.

## What changes

- `configengine` gains a `machine` layer for hub-wide modules: a never-tracked, sparse file `<BoardDir>/.lyx/config/<module>.machine.yaml`.
  The stack is template, default, machine, local.
- A new key category, `CategoryMachine`, opens a key to the machine layer and keeps it closed to the worktree-local layer.
  `MachineKeys(spec)` derives the open set; a machine document is validated like a local one and fails with `ErrInvalid` naming the file and the key.
- Gate's `slots` and `go_parallel` are reclassified to `machine`; `busy_quit_sec` stays hub-wide.
- `lyx config <module> --layer machine` works for `--print`, `--set` and editor writes.
  `--resolved` names `machine` as a unit's layer, and the menu marks a module whose machine file exists.
- Gate load failures (`gateslot`, `hubgeom.GateSlots`, `lyx gate test`) name both files and both ways forward when the machine file is in the stack.
  They also say that a session that is not the prime takes none of these steps and reports status: FAILED for the orch to run.
- Docs: `internal/configengine`, `internal/configcli`, `internal/gatecli`, `internal/gateslot` and `internal/hubgeom` `doc.go`, `docs/overview.md`, `PATTERN.md` and the `PATTERN-config-layers`, `PATTERN-hub-wide-config` and `PATTERN-durable-vs-ephemeral-state` background files.

No GitHub issue is attached to the task, so there is no `Closes` line.

## Design decisions

- **Where the file lives.**
  The machine file sits in the board checkout's untracked `.lyx/`, which is machine-local by construction, and the gate slot pool already lives in the same checkout.
  Every worktree of the hub on a machine therefore reads the same file and agrees on the pool size.
- **When the layer applies.**
  Only when the spec is hub-wide, has a `machine` key, and both `Dirs.Board` and `Dirs.Anchor` are set.
  A worktree whose anchor has no `_lyx/` still stacks the machine layer, since the rule is judged on the told `Dirs` before `load` and `Provenance` clear the anchor.
- **Hub-file checks stay machine-free.**
  A load with `Dirs{Board}` alone (hub-file validation, `CheckLoad` around a hub-wide `--set`, the hub-wide editor save) stacks no machine layer, so a machine override cannot mask an invalid hub value on its way to Bolt.
- **Writes are prime-only.**
  A machine write passes the same `hubWriteRefusal` guard as a default-layer write, is never committed, runs no fabric sync, is checked afterwards by `CheckLoad` over the prime's stack, and is put back on failure.
  The guard stops a mistaken write by a pair or slugged session, not a direct file write.
- **Never reconciled.**
  No reconcile, `hubreconcile.Ensure` included, reads, writes or removes a machine file: the board-side reconcile commits what it writes and rolls back only `_lyx/config/`, and an `Ensure` failure would block preflight, arm, orch start and clone.
  A retired key recorded as `machine` is admitted and dropped silently; any other stale key fails the gate load with the message that names both files.
- **Refusals name a way forward.**
  `--layer machine` on a per-worktree module names `--layer local`; on a hub-wide module with no `machine` key it names `lyx config <module>` from the prime.
  A per-worktree module declaring `CategoryMachine` fails a `configreg` registration test.
- **Limits.**
  `slots` and `go_parallel` still must be at least 1 and have no upper bound.
  Lowering `slots` takes effect on the next acquire, because `Pool.Limits` is read at every acquire.

## Known risks

- Two `lyx` builds on one machine and hub (a `.dev-bin` build and the installed one) read the same machine file.
  A build without the layer ignores it and may disagree on `slots` until updated.
- A forgotten machine override makes a hub-value change on Bolt not take effect on that machine; `--resolved` and the menu mark show it.
- `gate-language-neutral` plans to replace `go_parallel` with a general `parallel` key; that move must classify it as `machine`, and carrying the machine value over needs an amendment to the never-reconciled rule.

## Check by hand

None of the following was run by hand.

- `pattern-durable-state-index-clause-dropped` and `pattern-push-both-sides-index-clause-dropped`: the `PATTERN.md` index lines for the durable-state and push-both-sides invariants had a clause dropped while the file was shortened to meet its size line.
  Later commits restored the clauses; confirm both index lines in `PATTERN.md` still carry them and the file meets its size line.
- `card1-machine-route-bound-missing`: check that the bound on how the machine route reaches the layer (Anchor-and-Board rule, no machine layer for `Dirs{Board}` alone) is stated where the code applies it, in `internal/configengine/layers.go`.
- `long-cap-unbudgeted-cmd-lyx-unverified`: the long-running cap on the `cmd/lyx` end-to-end path for the machine write is not budgeted or verified; run `lyx config gate --layer machine --set go_parallel=8` from the prime and check the file and that Bolt has no new commit.
- `validate-machine-covered-unit-refusal-wording`: check the wording of the refusal when a machine document holds a key that a covering unit refuses, in the machine validation in `internal/configengine`, for the file name, the key and a way forward.
- `gate-load-failure-no-pair-way-forward`: check that every gate load failure message names the step for a session that is not the prime, including the `GateSlots` wrapper and the way forward `lyx gate test` adds after a failed limits read.
