# Slow whole-repo checks leave task gates for landing and an upkeep run

The main command package's test binary holds most of the module's whole-repository guards.
It reaches about 2 GB while its reachability guard builds an SSA call graph of the whole program.
That package imports nearly every package, so nearly every change put it in the impacted set, and every task gate ran those guards.
The tests that build a real .NET project added minutes to `internal/gatecli`'s runs.
None of these needs to stop a task's own gate: a break they catch can wait for landing, and dead code or a .NET break can wait a few landings.

## What changes

**A test can name the gate keys that run it.**
A top-level test whose directive lines carry an `only <key>...` marker runs only in a `--gate` test call whose key the marker names.
When the gate config derives scopes from Go packages, every `--gate` call scans its module for the markers (`impactset.FindOnlyTests`, `ModuleOnlyTests`).
The call holds back each marked test its key is not named in:
every listed tier run adds it to its `-skip` beside the watched tests, and the guard step leaves it out.
A call whose words after `--` carry a filter flag holds nothing back, and so does a call without `--gate`, so a hand run reaches every test.
A marker naming a key `gate.gates` does not hold refuses the call, naming the test and the key.

**A gate key of scope `marked` runs only the tests marked for it.**
Its listed tiers run no package, and its guard step runs each marked test under its file's tier, the way guard tests run.
The template gains the key `upkeep: {tiers: [0], scope: marked}` (`gateslot.UpkeepGateKey`, `gateslot.ScopeMarked`).

**The green record keeps what a pass held back.**
A green entry records the held tests in `held`, apart from its args.
It covers a later request, or serves as a computed scope's base, only when that request holds back each of those tests too.
So a webster gate that skipped the landing checks never lets Finalize carry its pass over them.

**Where each check now runs.**
- Every guard of the main command package that carries no watch marker now names `parent-align finalize` in its only marker.
  So do the tree-sitter link check and the Linux cross-compile.
  The task gates skip them, and the landing gates run them once over the whole tree.
- The reachability guard (`TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain`), `TestGateRedFirst_Dotnet` and `TestGateTest_DotnetFixture` name `upkeep`.
- The prime's dobby, whose worktree is on the main branch, now gives its board-tidy agent an upkeep step (`Deps.UpkeepCall`, the new `beam-fragment-board-tidy-upkeep` stencil).
  The step runs the `upkeep` key and files a tracker note per failure:
  one per function a dead-code test names, and a `bug` note for any other failing test.
  A beam's agent gets a step that runs no upkeep checks.

**The computed and `--base` guard step closes a gap.**
It used to leave every guard test of an impacted package to the listed runs, so a guard of a tier no listed tier runs ran nowhere.
It now also runs such a test, as the `--card` guard step already did (`Targets.InsideGuards`, `skippedInside`).

**The `create-caller-later` plan check is retired.**
It refused a card that creates a production function only a later card calls, because that card failed its own gate on the reachability guard.
The guard no longer runs at a card's gate, so the check, its spec entries and the plan-card rule it backed are removed.

## Design decisions

- The marker names gate keys, not an abstract site that each key would list in config.
  So existing hubs need no config change, and a hub with no marked tests behaves as before.
- The landing checks name both `parent-align` and `finalize`.
  With the same held set at both keys, Finalize carries Parent-Align's pass over an unchanged tree instead of rerunning every tier.
- A watched guard stays at the task gates: it reads a bounded set of files and runs only when one of them changes, so it is no whole-repository check.
  Guards of packages other than the main command package stay where they are too.
- `TestGofmt_ModuleFilesAreFormatted` moves to landing with the other unwatched guards, so a formatting slip is first caught at Parent-Align.
- A module whose packages `go list` cannot list holds nothing back and says so, so its tiers still report the break; a `marked` key refuses instead, since it would run nothing.
- The template's `upkeep` key names tier 0 alone, the tier every hub declares.
  The upkeep tests of this repository are tier-1 tests, so this hub's gate config needs `gates.upkeep` set to `{tiers: [0, 1], scope: marked}`.
  Until then the upkeep run prints a line naming the tier-1 tests it does not run.

## Merge danger

**Door:** two-way
**Blast radius:** gate

A revert restores the old skip and scope behaviour.
Green entries written meanwhile carry a `held` list that the reverted binary ignores, so on an unchanged tree such an entry could carry a pass that skipped the held tests.
That lasts until the tree changes.
