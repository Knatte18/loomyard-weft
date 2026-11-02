# Fixer-report, code round 1

## Changes

- **F1** fixed as proposed (`e10bb751`): `revision:` in the new benchmark section is now `e1770b2a`; the only later commit changed comments.
- **F2** fixed differently (`e10bb751`, reworded after the self-check in a follow-up commit).
  The proposed change, clock times per run or a checkable slot-holder window, was not made: no clock data for the runs is held in the worktree, and inventing it would be false.
  The "about 14:24 to 14:30" claim is removed instead, and the text says the overlap is not checkable from the report.
  The 8704-file and 35 MB figures now name their source, the two-second `find -type f` and `du` samples.
- **F3** fixed as proposed (`e8b427ab`): `runNameDigits` is now `runNameLength` at its declaration, in `randomRunName` and in its doc comment; the "over a million names" line is deleted.
- **F4** fixed as proposed (`72a3d517`): the widest-uid row uses `/tmp/lym-4294967295/r0a1b`.

`go build` and `go vet` of `./internal/lymtemp` and `./internal/testkit/tmuxkit` pass.
No test was added; no finding was a coverage gap.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: the reworded F2 sentence said the default run "waited for the shm run's slot" and also "ran alone after the shm run's start"; fixed as proposed by stating only what the table shows.
- fix-proposal: no findings; it noted F2 as fixed differently, recorded above.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
