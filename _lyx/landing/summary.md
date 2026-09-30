# loomcli: one driving surface per run, chosen by the recorded driver

A loom run's tmux session now shows one driving surface.
An `llm`-driven run shows the ly-drive strand and no `loom-status` strand;
a `go`-driven run shows the `loom-status` strand, as before.
The status band above ly-drive was noise, since the driver already reports progress.
`lyx loom status` stays available as a command for both.

## What changes

- `lyx loom start` brings reed up unconditionally, then branches on the driver `seedAndCommitBootstrap` already returns.
  On `go` it runs today's keep/replace/add `ensureStatusStrand` logic unchanged.
  On `llm` it calls the new `removeStatusStrands`, which clears a band left by an older build or an earlier start.
- `lyx loom step` and `lyx shed step` call reed `Up` in place of `ensureStatusStrand` and do no strand work.
  Before, every step of an llm run re-added the band through `lyx shed step`.
- `ensureStatusStrand` no longer calls `Up`; its only caller is `start`.
- `start`'s long help, `docs/overview.md` and the comments in `sharedbootstrap.go` and `internal/shedcli/parity_test.go` describe the new behaviour.
- `CONSTRAINTS.md`: the Driver Choice Single-Site Invariant now says the branch selects the run's driving surface (which driver spawns and whether the session carries the status strand) and nothing else.
  The driver is still read once, in `start`, so the tripwire test needs no new carve-out.

## Design decisions

- `removeStatusStrands` removes every strand named `loom-status`, not the first, because an older build may have left a duplicate.
- Removal is non-recursive.
  Reed refuses a non-recursive removal of a strand with children, so such a strand stays up with a warning rather than cascading through strands this change never meant to touch.
- A failed status read or removal logs a warning and does not fail `start`, matching the existing stance on a failed `ReplaceStrand`: a leftover band costs screen rows, not the run.
- `step` stays driver-blind, so it does not clear a leftover band.
  The next `start`, the resume path, does.
  An unconditional removal in `step` would also strip the band from a halted go run an operator steps by hand.
- Any driver value other than `llm` (empty included) takes the go arm and gets the status strand, per `mustUseLLMDriverArm`.
  Unseeded worktrees are seeded `llm`, so fixtures that relied on the band now seed `go` explicitly.
- No pane-height change: with the band gone the llm stack is ly-drive over the agent, and the hub's `collapsed_rows` already fits.

## Known limits

- A go run pre-seeded and driven only by bare `lyx loom step`, never `start`, gets no status strand.
- A running ly-drive session keeps its leftover band until its next `start`.

## To check by hand

- Start an llm-driven run in a session that still holds a `loom-status` strand and confirm `lyx loom start` removes it, then confirm several `lyx shed step` calls do not bring it back.
- Start a go-driven run and confirm the status strand still appears.
- `MalformedStatusProceedsToHandoverAndLogsWhy` and `DiedDriverProceedsToHandoverAndLogsWhy` in `internal/loomcli` fail when `TMUX` is set in the environment.
  Whether they already failed on `main` was not checked.
