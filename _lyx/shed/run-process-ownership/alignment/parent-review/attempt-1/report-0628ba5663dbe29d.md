---
verdict: APPROVED
---

## Scope against the board entry

The design covers every goal point of the entry `run-process-ownership` and its three notes.

- Owner record for every run process, listed by `lyx tender status`: D1, D2, D8.
- `reed down`, `fabric remove` and batten teardown end the pair's processes: D4, D5, D9.
- `fabric remove` does not wait on a paused, done or landed run's driver, and names what it ends: D5 (`ended_processes`).
- A queued push under a pause becomes a record, not a live process, and goes with a removed pair: D6.
- No archive tag of stale records from a machine that does not own the run: D5's local mode for `owned_elsewhere`, on top of `0d953f140`.
- Orphans of a removed pair are reported and ended: D8 (`lyx tender reap`).
- Note `pause-background-work`: hibernate confirmation treats shells as settled, lets forks finish within a bound, and leaves out the requesting orch (D10).
  The Webster fork context bound stays out, as the operator called it a separate idea.
- Note `review-waits-background-shells`: a gated row is not held by non-awaited shells, and an ungated shell-only wait surfaces as a stall (D11).
  The PATTERN-search question stays out, as the note itself says.
- The diverged Bolt and prime weft point is left to `e610f9250`, whose records replay and `lyx fabric pull` way forward settle it.

Two departures from the issue comment's "what should hold" are named in the record with their reasons, and neither is a direction the entry rules out:

- D6 keeps the driver chain parked under a pause.
  The entry's Expected list allows this ("or `fabric remove` ends them itself"), and D4/D5 make every end verb reach the parked chain.
- D2 lets a detached child outlive an interrupted CLI call.
  Detached pushes exist so that no command waits on the network (`fabric-remote-async`, operator decision 2026-10-18); synchronous children stay tied, and D6 stops the incident's case of a push parked under a pause.

"One verb ends them all" is split between `batten-cancel-verb`'s verb over `runproc.End` for a live run and `lyx tender reap` for survivors of a removed pair.
That matches the operator decision of 2026-10-22 recorded on both entries.

## Fit with other board entries

- `batten-cancel-verb` depends on this task and owns the verb that ends a batten run for good; the design builds only `runproc.End` for it and leaves the killed batten's record to it (D4, Scope Out).
- `spawn-audit-log` is left out; the design names D2's registration sites as the ones it extends, so no second launch path is set up.
- `orch-strand-cycle` depends on this task and changes reed's strand liveness; D9 changes reed's pair-session end, which is different behaviour.
- `fabric-remote-async` rules that no command waits on the network.
  D5 adds the run-scope origin reconcile to `fabric remove`, which the landed `PATTERN-origin-reconcile` requires, and which is the same kind of origin read the note keeps synchronous; the record states and resolves the tension.
- `verify-watchdog` (Windows tree kill in verify) is not touched; D3's Windows bound concerns only `runproc.End`.
- `umbrella-daemon`'s dobby will read the owner record later; nothing here builds dobby.

## Overlap with runs in flight and parent commits

`lyx loom peers` lists no parent commits.

- `finalize-dirty-names-paths` (row 6, own row 3): its design changes Finalize's dirty-merge stuck reason, `MergeGuardError`'s fields, the halt note's parent fabric status section and the loom/darn goto guard.
  This run touches none of those behaviours or their PATTERN entries; any shared package, such as `internal/loomcli` or `internal/fabricengine`, is changed at different places.
  No design conflict.
  Landing order: `finalize-dirty-names-paths` is further along and lands first; this run takes its changes at Merge-In.
- `board-entry-handles` (row 0): its design changes board handles, task priority and README layout in `internal/boardengine` and `internal/boardcli`.
  This run changes no board behaviour.
  No overlap; order unknown, since the peer's row is 0.
