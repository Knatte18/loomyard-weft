# Commit a batten run's start rows without a records push

A batten run pushed its records to origin before every transition, so each start row from Seed-Child to Run-Shed waited 10 to 20 s on a push.
A run start took 40 to 70 s since `71d6ee1a0`, against 1 to 13 s before it, measured from the `batten: <row> -> running` commits on the records branches.

## What changes

Batten's status-commit seam (`internal/battencli/commitstatus.go`) now commits a start transition without pushing it.
A start transition enters a row from Seed-Child to Run-Shed in state `running`,
and the seam's last-commit marker holds a `running` transition at a row before Run-Shed.
So the uninterrupted chain Worktree-Create → Seed-Child → Open-Window → Seed-IDE → Open-IDE → Run-Shed pushes once, at Worktree-Create.
The first transition after it, such as Land, or a halt or pause at Run-Shed, pushes and carries the start's commits.

Every other transition pushes as before:

- Worktree-Create, which has no marker yet;
- a start row resumed from a pause or a halt, whose marker holds that halt state;
- a start row that halts or pauses;
- every row after Run-Shed, and Run-Shed itself once the run has left its start;
- any transition whose marker is missing or unreadable.

The pushless commits go through a second `statuscommit` core whose push does nothing and which takes no gate admission, since it makes no network call.
The no-op-transition skip, the mid-merge skip and the commit's hard-error disposition are unchanged.

## Takeover

A machine that adopts the run before its first later push reads the run at its last pushed transition, Worktree-Create running.
It runs the start rows again from there.
The starting machine's next push is then rejected, reads the takeover entry and hands the run over through the existing `pairteardown.HandOver` path.
A catch-up after an absence still pushes the unpushed start commits, since it reads any local commit origin lacks.

## Design decisions

- Only a start row reached straight from a running pre-Run-Shed row skips the push.
  Run-Shed's resume after a pause therefore pushes:
  the inner run is already going then, and a takeover must be seen before Run-Shed watches it again.
- The rule lives in the batten seam alone; `internal/statuscommit` and the loom seam are untouched.

## Docs

`PATTERN.md`'s `PATTERN-run-fsm` entry and `pattern/PATTERN-run-fsm.md` state the exception and gain a "The start rows commit only" section;
`pattern/PATTERN-pair-remote.md` points at it.

## Tests

`TestNewCommitStatusSeam` in `internal/battencli` gains a pushes count per row and rows for the full start chain, a start with gated rows skipped, a resumed start row and a halted start row.

## Merge danger

**Door:** two-way
**Blast radius:** batten
