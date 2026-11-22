# Run a verify gate's reports alongside its tests

A `lym gate test --gate <key>` call used to run the key's report steps (reach, redundancy, mutation) one after another once its tiers and lints had finished.
The darn slowdown measurement (`docs/research/darn-slowdown-measure.md`) put that tail at about 75 to 105 s per verify gate.
The steps now start beside the first run of the plan's highest tier and judge in the background while the rest of the plan and the lints run.
The call writes the reports after its lints, so the report files and the printed lines keep their place and order.

## Behaviour

- The steps start only when no earlier run failed.
  A run that fails after they started cancels them, and the call judges red-gate reports as before.
- A halted call cancels the steps and writes no report.
- The steps' own lines, and those of their memory guard, are held until the reports are written, then print in step order before each report's.
- Each step still takes one gate slot where it builds, but now beside the slots the last tier holds, not after their release.
  A pool too small for both makes the step wait for the tier's release.
- Reports stay advisory and never change the exit code.

## Design decisions

- The "concurrent with tier 2" option was chosen over "only on the last verify gate", so every gate key that lists reports gains the overlap and no gate loses its reports.
  Red-first is untouched.
- The report steps' work would otherwise trip the tier run's guards, which watch the same process tree:
  - The tier's process cap (`processcap.go`) counts a marked process only when it sits below a running tier command.
    The new `runningCommands` records those commands, and `testRequest.commands` carries them.
    A marked process whose parentage the process sample lost still counts, so a tmux server that left the tree stays counted.
  - The report steps' memory guard (`memoryguard.go`) leaves alone a test binary below a running tier command, which the tier's own guard watches.
- The orchestration lives in the new `internal/gatecli/reports.go`: `reportStepsOf`, `reportChain` (start, cancel, stop, wait) and `judgedReports`.
  `reach.go`, `redundancy.go` and `mutation.go` split their steps into judging and writing so the write waits.
- `lym gate test` docs, `gateslot` docs, the `gates` template comment and `pattern/PATTERN-test-speed.md` describe the new order.

## No timing in this change

The entry asked for one verify gate timed before and after.
That bound was dropped for this run: an after-timing needs a `lym` built from this branch, which no step of the run builds.
The gain is to be measured from archived run records after landing, so this description states no wall times.

## To check by hand

- `internal/gateslot/doc.go` and `pattern/PATTERN-test-speed.md`: confirm no sentence still describes the report steps as running after the tiers or in sequence.
- `internal/gatecli/memoryguard_test.go`: confirm a test pins the report steps' memory guard being wired with the foreign-binary check (`foreign`), not only `withoutForeign` alone.
- `internal/gatecli/run.go`: confirm a test pins `lastTierStart`, the plan index where the report steps start.
- `internal/gatecli/processcap.go`: confirm the report steps' process sharing the last tier's cap cannot fail a tier, and that `processcap_test.go` pins it.
- Time one verify gate of the same diff before and after, to confirm the saving the research measured.

## Merge danger

**Door:** two-way
**Blast radius:** gatecli
