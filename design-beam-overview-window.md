# The overview window becomes the beam window

`beam-overview-window` · Task · enhancement, batten, orch, reed · running · Code-Bouncer

## Seen (operator, 2026-10-25)

The overview window, the `runs` strand running `lyx batten overview`, shows the orch's context line, the runs in flight, and since `dobby-overview-window` the worktree's dobby with its runs, so the runs show in two places.
Its name, overview, says neither that it belongs to a beam nor what it covers.

## Design (operator, 2026-10-25)

The overview window becomes the beam's own window, named `beam: <slug>`, `beam: prime` in the prime, with one section per part of the beam:

- Dobby: alive, `max_runs`, each run it drives with its state and stage, and each released task it has not started with what it waits on.
- Orch: its context reading against the soft threshold and the hard cap, and its cycle state, as the context line shows today.

Each run shows once, in the dobby section.
A beam's window shows its own orch and the orchs of every beam below it.
