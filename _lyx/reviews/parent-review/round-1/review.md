---
verdict: APPROVED
---

The design does what board entry `scratch-defeats-test-cache` (issue #771) asks: it keeps `.scratch/` churn out of what the source scans read and Go keys its test cache on.
The entry offers two remedies, and the design takes the first; the second, documenting that scratch files be created before the first gate run, is recorded as a rejected alternative with a reason.
The entry carries no separate Done-when; the acceptance criteria cover the goal, including the cache-relevant `Lstat` stability of the root `.scratch` entry.
Nothing in the design belongs to another entry: the Windows junction behaviour is left to `fabric-windows` as an open risk, and dirty-tree reporting stays with `finalize-dirty-names-paths`.
The target `scratch` under the weft-side `.lyx` does not collide with any existing `.lyx` subdirectory (`config`, `webster`, `shed`).

## Overlaps

- `origin-reconcile` (row 11, `running · Webster`): both runs change files in `internal/fabricengine`, including `doc.go` and the mutation record.
  That run's origin reconcile, records replay and push retry do not touch scratch wiring, the warp exclude or `repairPairWiring`'s junction branch, so the overlap is text-only.
  Landing order: `origin-reconcile` is further along (row 11 against own row 3) and lands first; this run takes its changes at Merge-In.
- `shellguard-fixes` (row 0, `running · Darn-Write`): its new redirect row treats a relative path under `.scratch/` as a scratch place.
  This run keeps agents writing to `.scratch/<name>` at the worktree root, so the two agree.
  Landing order: unknown, since the peer's row is 0.
- Parent commit `0d86509` (`webster: skip create-not-done for a root .scratch/ Create target`): exempts a root-relative `.scratch/` `Create` target in `planglyph.DoneChecks`.
  This run keeps that path, and a file reached through the link keeps its root-relative spelling, so the exemption still holds.
  Landing order: already landed on the parent; this run takes it at Merge-In.
- `gate-machine-local-config` (row 7) and `run-reentry` (row 9) mention `lyx fabric reconcile` or scratch files only in passing and change no behaviour this run changes; no overlap.
