# Fixer-report, round 1

## Changes

- **F1 (MEDIUM), fixed as proposed, then extended after the self-check.**
  `surfaceBeamUpWarnings` takes an injectable `notify` function, which `pawlDeps` builds once over `orchcli.NotifyOrchAt` with `pawlProgram` and shares with `Deps.Notify`.
  The Beam-Up step's body is now `runBeamUpStep`, which takes the bring-up as a function.
  `TestSurfaceBeamUpWarnings` has rows for no warnings, warnings and a failed post;
  `TestRunBeamUpStep` pins that the step surfaces the report's warnings, that a failed post never halts the row and that a bring-up error halts it with no notice.
  Commits `96260b38` and the last commit of the round.
- **F2 (LOW), fixed as proposed.**
  Two rows in `TestStart_EnsuresTheRunsStrandBeforeTheOrch`, `loom-status` before a stale `runs` strand and after it, each expecting a replace of the runs strand's guid.
  Commit `07f86524`.
- **F3 (LOW), fixed with my own change.**
  The proposal extended `TestSmokeStatusStrandAcrossDriverSeeds` over a real reed session; I added a sibling subtest on a fresh pair instead, because the existing pairs already hold a status strand and a start there keeps or replaces it, which would blur what the row pins.
  It adds a runs strand, runs `lym loom start --no-attach` and asserts the status strand sits in a window other than the runs strand's.
  `statusStrandPane` now calls a new `strandPane` helper.
  The tier-2 run executed the subtest and it passed.
  Commit `c88d97c6`.
- **F4 (LOW), fixed as proposed.**
  `docs/overview.md` lines 421, 539, 677 and 811 follow the wording of the updated `internal/loomcli/doc.go` and `internal/orchcli/doc.go`.
  Commit `09267368`.

## Self-check

- fix-code-correctness, LOW: the failed-post row cannot fail on the property its name states, and the step closure is unpinned. Fixed with my own change: `runBeamUpStep` and `TestRunBeamUpStep`.
- fix-code-correctness, note: the tier-2 run's skip count hid whether the F3 subtest ran. Checked in the gate log: it ran and passed. No change.
- fix-consistency, LOW: the smoke file's header did not name the new subtest's property. Fixed.
- fix-consistency, LOW: the doc comment of `TestSmokeStatusStrandAcrossDriverSeeds` said one pair per driver seed only. Fixed.
- fix-proposal, MEDIUM: F1's proposal pinned the helper, not the step's wiring. Fixed with my own change, as above.
- fix-proposal, LOW: the F3 subtest's visibility in the gate runs. Checked in the log as above. No change.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
