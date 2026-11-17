# Fixer-report, code round 1

## Changes

- F1 (`81ae5dc2`): `TestWriteHold` gained a row that seeds a hold under a machine's former name, registers the rename, and asserts `WriteHold` under the new name writes nothing, returns no `Standing` and leaves one hold on origin.
  The hold is seeded through `WriteHold` before the rename, since a second clone would be stale after the registrations.
- F5 (`81ae5dc2`): pinned the contract as it is: a hold of the same machine with another kind is appended beside the first.
  A `TestWriteHold` row asserts both stand on origin; the `WriteHold` doc comment and the `boardengine` package doc state it.
  The review's alternative, one hold per machine, was not taken (see Deferred).
- F2 (`a6e2f912`): `UpgradeDeps.writeHold` posts the notice only when `Wrote` is true or `Standing` is non-nil.
  A `TestUpgradeRead` row finds this machine's identical hold standing and asserts no notice and a dropped entry.
  The row is a table row over a new `ownHoldStands` hub field rather than a second run, since the hub fake already stands for the found-standing answer.
- F3 (`93684c3f`): `SyncBoltHold` lifts a bolt-hold entry that records another machine than `req.Machine` and writes a fresh one through `Pause`.
  A `TestSyncBoltHold` row covers the move, and the machine check compares each entry to its request's machine.
- F4 (`6e44c322`): the `OwnerCaller` comment and the owner paragraph of the `pauserecord` package doc say the name stays declared only so the upgrade's first read recognises and drops legacy entries.
- Follow-up from the self-check (uncommitted at report time, committed with this round): `internal/tenderengine/doc.go`, `internal/dobbyengine/doc.go`, `internal/boardengine/doc.go` and the `SyncBoltHold` comment state the F3, F2 and F5 behaviour their twins left stale.

Gate: `lym gate test` tier 0 over `boardengine`, `dobbyengine`, `tenderengine` and `pauserecord` passed; tier 1 over `boardengine` passed.

Gate repair: three comment lines the round added broke the line rules, so no test ran.
They were split at sentence boundaries in `internal/pauserecord/doc.go`, `internal/tenderengine/doc.go` and the `SyncBoltHold` comment in `internal/tenderengine/bolthold.go`.
The `SyncBoltHold` comment returned to its earlier opening sentence, since its replace sentence already states the behaviour.

Gate repair, second: `TestGofmt_ModuleFilesAreFormatted` failed on `internal/dobbyengine/upgrade_test.go`, whose struct-field alignment the round's new `ownHoldStands` field broke.
`lym loom fmt` realigned it; no other file was unformatted.

## Self-check

- fix-consistency, MEDIUM, `tenderengine/doc.go` package doc says a slug with an own entry gets none: fixed as proposed.
- fix-affected-set, LOW, same `tenderengine/doc.go` text: duplicate of the line above, fixed.
- fix-consistency, LOW, `dobbyengine/doc.go` says one notice unconditionally: fixed as proposed.
- fix-consistency, LOW, `boardengine/doc.go` lacks the F5 line: fixed as proposed.
- fix-code-correctness, LOW, F3 replace is two record updates with a transient window: disputed, every fix worse (see Disputed).
- fix-proposal, LOW, F3 non-atomic replace and uncovered window: duplicate of the line above, disputed.
- fix-proposal, LOW, F3 raw machine-name comparison: disputed, premise false (see Disputed).
- fix-proposal, LOW, F2 notice lost on a cut between write and notice: disputed, every fix worse (see Disputed).
- fix-proposal, LOW, F5 second hold inert under `HoldOf`: deferred (see Deferred).
- fix-proposal and fix-affected-set, F1, F4: no finding.

## Disputed

- F3 window, every fix worse: an atomic swap needs a new `pauserecord` update path that rewrites an entry's `Machine` in place, which leaves `tenderengine` and adds a second writer of entries.
  The existing drop branch of `SyncBoltHold` lifts through the same `Lift`, so the window is the one the package already has.
- F3 raw comparison, premise false: `SyncBoltHold` stamps `Machine` from `req.Machine` through `Pause`'s `HoldMachine` (`bolthold.go`), and its one caller takes `req.Machine` from `owners.Holds[0].Machine` (`internal/beamcli/dobbydeps.go:123-138`), so both sides are the one recorded string and cannot flip between a current and a former name.
- F2 lost notice, every fix worse: posting the notice on an identical standing hold restores the duplicate the finding F2 removed, and `Notice` failures are only logged either way.

## Deferred

- F5, a second hold of one machine is inert under `Owners.HoldOf` (`internal/boardengine/registry.go:159`, which answers the first hold): whether a machine holds one hold or several is an operator decision on the hold model, and the review allowed either contract.
  This round pinned the current append behaviour; a change to one hold per machine also changes `HoldOf` and its callers.

## Rulings

None reached this round.
