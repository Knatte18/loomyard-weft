# Each run transition waits for a push to origin, so a run starts about a minute slower

`transition-push-slows-rows` · Note · enhancement, batten, fabric

## What was seen

The batten start rows, from Seed-Child to Run-Shed, took 1 to 13 seconds per run up to 2026-11-18 08:23, and 40 to 70 seconds per run from then on (per-day mean: 7 to 12 s on 11-15 and 11-17, 56 to 69 s on 11-18 to 11-22).
The jump sits exactly at `71d6ee1a0` (Adopt in the foreground over pushed state), whose member `595349b9f` pushes a transition's records to origin before the run acts.
Each row now waits for a push to origin of about 10 to 20 seconds; the start rows Seed-Child, Open-Window, Seed-IDE, Open-IDE and Run-Shed each pay it, and so does every other transition of a run, Land and Worktree-Teardown included.

Measured from the `batten: <row> -> running` commits on the records branches (`git log --all --grep`).

## Idea

Keep the pushed state that a takeover needs while cutting the wait per row:
push only at the transitions a takeover reads from, batch several transitions into one push, or push in the background and wait only before a row whose effect a second machine could repeat.
The design must show which transitions adopt needs pushed before the act.
