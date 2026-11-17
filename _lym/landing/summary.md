# Warn on resume when a run's stencils changed since it started

A run lasts hours to days, and stencils change often.
A resumed run's later producers could render prompts that differ from the ones it started with, and nothing said so.

A run now pins the body hash of every installed registry stencil when it is first seeded.
A `lym loom start` or `lym loom resume` that spawns or wakes a driver compares the pins with the installed board copies.
It posts one notice to the run's parent orch naming each pinned stencil that changed or is gone, and naming `lym stencil diff <name>`.
The success envelope lists the same names under `stencils_changed`.

## Behaviour

- The pins live in `stencil-pins.json` under the run directory and are committed with the seed.
  A re-entry keeps the record and commits it whenever it is on disk, so a first seed whose commit failed does not lose it.
- A stencil with no board copy at pin time is left out of the pins, since nothing can render it.
  A stencil installed after the pins were taken is never reported.
- A start that finds a driver already working resumes nothing and compares nothing, so the parent gets no duplicate notice.
- A run with no pins warns of nothing.
  A failed hash, read, compare or post is logged at Warn and never fails the verb.
- `resume` releases the bootstrap lock before the check, so the post never runs under the lock.
- The notice says `lym stencil diff` compares a stencil with its shipped default and its board copy.
  It does not claim a diff against the pinned version, which is stored only as a hash.

## Code

- `internal/stencilstore`: `Pins` and `ChangedPins` hash the board copies and report the changed names in name order.
- `internal/shedrun`: `StencilPins` and its read, write and path helpers.
- `internal/loomcli`: `stencilpins.go` holds the write and the warning.
  `runDriverSpawnAndWait` now also returns whether this call put a driver to work.
  The `loomCLI` struct gains a `notifyParent` seam, built over `orchcli.NotifyParent` and replaced by a recorder in tests.

## To check by hand

- The parent notice's wording against `lym stencil diff` (`notice-overclaims-stencil-diff`, `internal/loomcli/stencilpins.go`):
  confirm it promises no more than the verb delivers.
- A failed seed commit (`pins-left-uncommitted-after-failed-seed-commit`, `internal/loomcli/sharedbootstrap.go`):
  confirm the pins record is committed on the next entry.
- The seed commit of the pins (`seed-commit-of-pins-untested`, `internal/loomcli/sharedbootstrap.go`):
  confirm a test covers the record being committed with the seed.
- The stencil check on start (`start-stencil-check-untested`, `internal/loomcli/start.go`):
  confirm tests cover a start that spawns, one that wakes a parked driver, and one that leaves a working driver alone.
- The change was not run against a live run with an edited stencil, so check one by hand:
  start a run, edit a stencil in the board's stencil directory, resume, and read the notice.
