# Cut fabricengine tier1 test time at least in half

`fabric-git-free-core` · Task · enhancement, fabric, testkit · draft

## Why

`internal/fabricengine` sets the length of most tier1 gate runs.
In `dotnet-gate-fixes`' verify on 2026-11-26 the tier1 run took 250 s, and the package alone took 211 s of it; the next package took 64 s.
The package holds about 240 test files and 630 top-level tests, 400 of them in integration files.
It also enters the diff scope of changes far from fabric, because its tests import many other engines (webster, weaver, drift, landing, hardener, claude and more), so a change to `gateslot` ran it.

## What to do

- Cut the package's tier1 cost by at least half, measured the same way before and after: the package's tier1 wall and serial time, and its git process count.
- Start from `docs/research/speed/fabric-test-speed-survey-2026-11-03.md`.
  Its levers 1, 3 to 6 have landed since; take what remains, chiefly lever 2 (`worktree list` read through `dotgit`, about 15 percent of git processes) and lever 7 (go-git for the code side's reference reads).
- Delete or fold tests that pin nothing another test does not, by the test-economy rules; a test that only pins plumbing becomes a cheap unit test or goes.
  Each removal states what it pinned and which test still pins it.
- Cut the test imports that pull `fabricengine` into the scope of changes to other engines, so a change elsewhere stops running it.

## Constraints

- No production behaviour change beyond the cheaper reads of levers 2 and 7.
- Report the before and after numbers in the change description.
