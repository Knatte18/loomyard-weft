---
verdict: APPROVED
---

The design covers the board entry's goal and each of its five design points, and does no work that belongs to another entry.

- Point 1, an owner per entry on Bolt shown by the board, batten status and tender status, with the prime carrying one: D1, D2 and D9; a machine starts, resumes and lands only what it owns: D3 and D7.
- Point 2, `lyx adopt` over named tasks, the prime or `--all`, the prime taking itself only, bringing repos up to date, adding missing pairs, recording the owner, pushing and resuming: D5; the wake and orch-start pause of a run owned elsewhere: D4 and D6.
- Point 3, landing only on the parent's owner, halting before the parent-side merge with two ways forward: D7.
  D7 words the second way forward as `lyx adopt <slug>` on the parent's owner, because the entry itself rules the owner-lands-without-takeover handover out of this task and into beam work.
  That stays within the entry.
- Point 4, the owner outcome in the origin reconcile: D4.
- Point 5, Open-Window per machine (#742): D8.
- Issue #741's orch reading a missing pair as landed: D9's stencil reading.

Scope Out matches the board: beams and the ready-to-land handover stay in note `orch-any-worktree`, process ending and pause leaks stay in `run-process-ownership` and `tender-pause-holds-reprompt`, and a local warp branch's adoption stays in `fabric-add-adopt-local`.
`run-process-ownership`'s own point that `fabric remove` should not push an archive tag from a non-owner machine stays with that task, which builds on this run's setter and seam.
The field name `owner_machine` departs from the `owner:` example in the entry and in `orch-any-worktree`'s operator decision; that is a naming detail chosen to avoid the strand and process owners those entries plan, and it contradicts neither.

## Overlap with runs in flight

`lyx loom peers` lists `own_row` 3 and no parent commits.

- `darn-verify-fixes` (row 5): its decisions change Darn's verify-failure routing at Merge-In and Publish in `internal/landingshed`, `internal/loomshed` and `internal/shedadapters`; this run adds an owner check at Finalize in the landing producer.
  Different behaviour, at most a text-only overlap in the same packages, and no shared PATTERN entry.
  Landing order: `darn-verify-fixes` is further along and lands first; this run takes its changes at Merge-In.
- `quarry-csharp` (row 3): plan parser and quarry glyph work; no overlap.
  Same row, a tie, which matters only if an overlap appears later.
- `research-in-discussion` (row 0): Discussion and Plan stencils and rubrics; no overlap; unknown order.
- `decision-add-notifies` and `shuttle-open-agent-defaults` (row 0): no decision record yet; their titles concern decision notices and shuttle's agent defaults, neither of which this design touches; unknown order.

This run adds a new PATTERN entry, `PATTERN-owner-machine`, and no peer changes it.
