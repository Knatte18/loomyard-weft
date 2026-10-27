# Fixer report — plan, round 1

## Changes

F1 is `decision`-class and left out here.
The parent's recorded decision "Adopt seed only for the hub gate.yaml" was applied to card 11 as the parent's instruction:
`MigrateConfig` stays unchanged and never seeds `adopt`;
`configreg.Module` gains `MigrateHubBase`, set on the gate row to a new `gateslot.SeedAdoptGate`;
only `ReconcileHubWideAt`'s present-file branch runs it, after `Migrate`.
A `TestMigrateConfig` row pins that `MigrateConfig` does not seed, `TestSeedAdoptGate` takes the seed rows, and `TestReconcileHubWideAt_SeedsAdoptFromATunedFinalize` pins the hub reconcile.
A hub `gate.yaml` seeded in a pass from the prime or legacy files gains `adopt` at the next reconcile, which the card states.
Card 14's "card 11's migration" now reads "card 11's hub-base seed of `adopt`".

- F2 (card 8): added a `TestRun_WorktreeGoneExits` row that removes the worktree after tick 1 has ended, from inside the `Deps.Sleep` seam, wakes the loop by one watch event with no probe due, and asserts `Run` returns nil, one `StopStrand` of the kept `Tidy`'s guid, and `CheckBinary` called only after tick 1. The self-check refined the row's mechanics.
- F3 (card 10): the row whose `wire` fake removes the prime's worktree link runs last, and the test description says so.
- F4 (card 7): `internal/beamcli/doc.go` moved from Extends to Edit, removing the Extends section. The Docs line says the registration-order sentence and the `already_running` sentence are rewritten, and that the placement refusal's sentence still holds.
- F5 (cards 13 and 15): fixed differently. The proposal put the `check.json` stat in `ReadCheckView` and had `ReadCheck` read through `state.ReadJSONNoMkdir`. The stat moved into `ReadCheck` instead, because `ReadJSONNoMkdir` takes a read lock whose open fails on a missing directory (`internal/lock/lock.go`, `AcquireReadLock`). That would make card 14's `ensureCheck` fail on a hub's first `lym adopt` (fork fix-consistency). `ReadCheckView` relies on `ReadCheck`'s absent read. A `TestRunCheck` row pins it, and `TestStatus_PrintsThePrimesDobbyAndEachLocalBeams` asserts the adopt scratch directory stays absent with no record.
- F6 (card 13): step 5 clears `Running` whether the record ends green or failed. A `TestRunCheck` row has a fake gate call `Started` and return 1 without `Ended`, and asserts the record ends failed with no `Running`.
- F7 (card 8): the kept-tidy refresh reads through `state.ReadJSONNoMkdir` on `Geometry.StatePath` and reads an undecodable record as absent. A read that fails while `worktreeGone` reports true exits through `exitWorktreeGone` with the earlier `keptTidy`, and any other read error ends the loop as a tick error does. `internal/state#ReadJSONNoMkdir` joins Uses.
- F8 (card 7): steps 1 and 5 write `dobby: stopped on error: <error>; way forward: fix the cause, then run lym beam start`, and the two `TestRun_LockOwnerAndWakeups` rows that assert the stop line assert the suffix.
- F9 (card 10): dropped the task-pair-without-beam and loose-directory rows from `TestRunDobbyWith_TracesEachExitBeforeRun`. `internal/beamcli_test#TestDobbyCmd_RefusesAPathItCannotPlace` joins Edit, and each of its rows also asserts no `dobby.log` and no `.lym/beam` directory under the root.
- F10 (cards 11 and 13): card 11 extends `pattern/PATTERN-test-speed.md` with an `adopt` Gate keys row. Card 13 edits `internal/fabricengine#AdoptScratchDir`, plus its twin `internal/lymdirs#AdoptDirName`, whose doc comment carries the same suite-log-only sentence, so both name the check's record, output and lock.
- F11 (cards 2 and 3): dropped "or returns an empty name" from card 2's refusal. Card 3's `TestMachineLandings` drops the empty-name row and its refusal half, leaving two rows.
- F12 (card 13): step 5 now reads "as on a refused key".

`lym loom validate-plan` returns ok.

## Self-check

- fix-claims: no findings.
- fix-consistency, MEDIUM, `ReadCheck` through `ReadJSONNoMkdir` errors on a missing adopt directory, which breaks card 14's `ensureCheck`: fixed with my own change. `ReadCheck` stats `check.json` first, and card 15's own stat is dropped in favour of it. Also recorded under F5.
- fix-affected-set, LOW, `docs/dotnet-hub.md`'s recommended `gates:` block lists every key and gains no `adopt`: fixed as proposed. Card 11 extends it with an `adopt` line.
- fix-proposal, MEDIUM, F2's row returns a watch event from `Deps.Sleep`, whose signature returns nothing: fixed as proposed. In that row tick 1's hook sends no event, and the wrapper removes the worktree on its first call only, sends one value on `Watch` and returns.
- fix-proposal, LOW, the new Gate keys row breaks the table's lead sentence that sites run `lym gate test --wait --gate <key>`: fixed as proposed. The row's Site cell says "in-process through `gatecli.RunGate`".

## Disputed

None.

## Deferred

None.
