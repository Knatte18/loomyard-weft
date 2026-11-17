```yaml
lens: fix-proposal
outcome: reported
```

- **[LOW] F3, `SyncBoltHold` replaces a bolt-hold entry by lifting it, then pausing again.**
  - Changed text: `internal/tenderengine/bolthold.go`, the `if hasOwn { ... Lift(...) }` block before `Pause`.
  - Summary: the replace is not atomic, and it leaves a window in which the run is uncovered.
  - Evidence: `Lift` goes through `liftEntries`, which runs `pauserecord.Update` once for the lift. `Pause` then runs its own `Update`. Between the two, the record has no live entry for the slug.
  - A session or network call that polls in that window reads uncovered and proceeds, until `Pause` writes the new entry.
  - The window is the same one the existing "left Hold" lift creates. The new risk is that the slug stays in `Hold`, so no one expects it to be uncovered.
  - If `Pause` fails after a successful `Lift`, the failure is joined into the error and the slug stays uncovered until the next tick. The finding's premise did not require that exposure.
  - A single `pauserecord.Update` that swaps the entry's `Machine` in place would avoid both.

- **[LOW] F3, a differing machine name forces a replace on every tick.**
  - Changed text: the condition `own.Machine != req.Machine`.
  - Summary: a comparison by plain string equality can loop, which is the class of bug F1 pinned for `WriteHold`.
  - Evidence: `boltHoldRefusal` treats an empty machine as possible, so `req.Machine` can be `""`. Suppose `Pause` stamps `HoldMachine` differently from the raw request value, or `req.Machine` flips between the hold's current name and a former name. Then each tick lifts and rewrites the entry, and each rewrite adds an entry to `Resumed`.
  - D3 requires machine-name comparisons to go through the registry. This comparison is raw string equality, so a hold written under a former name would read as moved.
  - I did not confirm that a caller can supply a former name here, so I rate it LOW.

- **[LOW] F2, the notice is lost if a cut falls between the hold write and the notice.**
  - Changed text: the `if written.Wrote || written.Standing != nil` guard in `UpgradeDeps.writeHold`.
  - Summary: the guard makes the notice at-most-once, where D5 needs it at least once.
  - Evidence: `Notice` failure is only logged, and the hold stands after `WriteHold`. If the process is cut, or `Notice` fails, after `WriteHold` returns `Wrote` true, the re-run finds the identical hold. It then returns `Wrote` false with no `Standing` and posts nothing. The operator never receives the way-forward line.
  - The reviewer's proposal swaps a duplicate notice for a possibly missing one. The duplicate was the safer failure.

- **[LOW] F5, the test pins `HoldOf` blindness to a second hold without noting it.**
  - Changed text: `AHoldOfAnotherKindFromTheSameMachineIsAppendedBesideTheFirst` in `holdwrite_integration_test.go`.
  - Summary: the pinned contract leaves the appended adopt-fix hold effectively inert.
  - Evidence: `Owners.HoldOf` (`internal/boardengine/registry.go:159`) returns the first hold that does not match `HoldKindAdoptFix` with `FixTask == slug`.
  - With `[adopt, adopt-fix]` standing, `HoldOf` answers `adopt` for every slug. So the fix task's own slug is held by the first hold, and the second hold's `FixTask` exemption never applies. The test asserts both holds stand and does not check what the reader sees.
  - The contract the test pins is therefore one under which the appended hold has no effect. Refusing the second hold, or replacing the first, is the reading the reviewer's alternative allowed.

- **F1: no finding.** The proposal holds.
  - The test seeds the hold under `neve` and registers the rename.
  - It then writes under `osl46`, asserting `Wrote` false, `Standing` nil, and one hold on origin.
  - It fails if `sameMachine` is reduced to `a == b`.

- **F4: no finding.** The proposal is wording only, and the retained readers still match the comments.

Further tool calls:
1. I read `Lift`, `liftEntries` and `HoldOf` with one grep. Context did not show `internal/tenderengine/resume.go` or `internal/boardengine/registry.go`.
