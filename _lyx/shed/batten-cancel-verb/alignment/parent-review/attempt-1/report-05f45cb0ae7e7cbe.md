---
verdict: APPROVED
---

## Scope against the board entry

The design covers every point of the entry's brief and its three folded notes, and adds nothing that belongs to another entry.

- One verb ending a run in any state, live or stopped, before or after its pair exists: D1, D2, D3.
- Ending its processes and strands so reed relaunches nothing, with no drive report, no stop notice beyond one cancel notice, and no two-minute quiet wait: D4, D9.
- Clearing the claim so the entry takes edits again, leaving body, scope and owner to the caller, as the `batten-abandon` note asks: D7.
- Removing records and scratch and committing and pushing the records side: D6, D9.
- The pair optionally, with `--remove`: D5.
- Batten refusing a dirty source up front, per the operator decision of 2026-10-20, with no record, pair or tackler: D10.
- Worktree-Create's dirty-source stop halting with its way forward and never spawning a tackler: D11.

The `batten-abort-live-run` note's idea of an `aborted` batten state is replaced by removing the records, which the original note and the `batten-abandon` note both ask for; the entry leaves the choice between the two ideas to this task, so it is not a deviation.
The `batten-abandon` note's refusal of a live run is superseded by the operator decision of 2026-10-22 that the one verb ends a run whatever its state.
The way-forward text changes in `boardengine` and `pairteardown` and the lift of the slug's own pause entries (D8) follow from ending a run for good and name the new verb; they take no other entry's work.
The process end goes through `internal/runproc`, the owner record `run-process-ownership` built, as the entry says.

## Other board entries

- `gate-runs-once` depends on this entry; its scope (test tiers, row entry gates) does not touch the cancel or the dirty-source check.
- `orch-any-worktree` and `pair-from-parent`: D1 keeps today's prime-only rule and D10 names the source worktree generically, leaving both entries' widening to them.
- `batten-window-default`: D10 checks before any path that opens a window, not only the `--window` flag, so it holds after that entry lands.
- `umbrella-daemon` and `batten-ready-overlap`: D7 states that a cleared entry reads ready and leaves holding it back to the caller; neither entry is designed here.
- `shed-arbiter-all`: D11's hand-back row sits ahead of the arbiter rows and only hands back; it does not change which runs have an arbiter.

## Peers and parent commits

`lyx loom peers` lists one peer, `plan-card-rules` (row 3, the same as this run's `own_row` 3), and no parent commits.
Its design touches the plan gate, `planparser`, `websterengine`, `tools/plandag`, `lyx loom decision add` and the Publish failure record; it changes no batten, shed driver, `pairteardown` or `fabricengine` behaviour and no PATTERN entry this run changes (this run changes only `PATTERN-pair-teardown`).
No overlap, so no landing order applies; had one existed, the two runs would be a tie by row.
