# Reopen a beam from PR-Review: cancel its pull request and go back to Beam-Await

`beam-reopen` · Note · feature, pawl, landing, cli

Imported from [issue #46](https://github.com/Knatte18/loomyard/issues/46).

A beam's pawl run cannot take new work once it has left Beam-Await. Suppose the operator flags a beam ready, the run reaches PR-Review, and then a new task should join the beam. Today that takes several untested manual steps:

- `lym loom goto --to Beam-Await` only moves a halted run, but at PR-Review the run is awaiting, so it has to be paused first and resumed afterwards.
- `lym beam ready` writes and commits `_lym/landing/beam-ready.json`, and no `unready` verb exists. While the flag stands, the dobby lands the beam again as soon as nothing works under it.
- The open pull request to the parent stays open with a stale body. Only lym may close it, and it is unclear whether a later PR-Review reuses it or opens a new one.
- The pr-reviewer's rewritten `summary.md` in the weft goes stale too.

This came up on beam `duplication-and-import-followups` (NORCE-DrillingAndWells/Models PR #267).

Wanted: one operator verb, for example `lym beam reopen <slug>`, that does all of this in one step:

1. Refuses unless the beam's pawl run is at PR-Review, or at a row after Beam-Await and before Finalize.
2. Stops the pr-reviewer and closes the beam's pull request with a comment saying the beam was reopened.
3. Removes the ready flag.
4. Moves the run back to Beam-Await and leaves it waiting, so the beam can take new members. A later `lym beam ready` lands it again through Parent-Align, Beam-Describe and a fresh PR-Review.

## Issue comment, 2026-11-04

Amendment to step 2: the verb need not close the pull request. It can keep the PR open, convert it to a draft, and post a comment saying the beam was reopened. The next PR-Review then reuses that PR, and Beam-Describe refreshes its body, rather than opening a new one. This keeps the PR's link, its review comments and its history. Closing the PR remains an option, for example through a flag, for the case where the reopened work changes the beam's scope enough that a fresh PR reads better.