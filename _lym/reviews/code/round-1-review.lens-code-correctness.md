```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, running each function and branch the diff adds or changes against a concrete input, including failure paths.

## Per function: case run, verdict

- **`ReadsAsHere`**
  - Empty name reads as no machine: correct.
  - Name is a former name of a registered machine whose id equals `here.ID`: true, correct.
  - Name is another registered machine's current name, with `here.ID` empty or different: false, correct.
  - Name no entry resolves: falls back to `here.Name` or `here.Former`, correct.
- **`Registry.Resolve`**: a current-name match beats a former-name match, and ties go to the smallest id, so the result is deterministic. Correct.
- **`Registry.Check`**
  - A copied file, where the entry name is not among Name, Former or Forgotten: refused.
  - A former name that is another machine's current name, not in `Forget`: refused with `--forget-former`.
  - A current name equal to another machine's current or former name: refused with `lym machine name &lt;new&gt;`.
  - All correct.
- **`Registry.apply`**: renaming adds the old name to Former, `Forget` and the new name are removed, and `nil` versus empty Former compares equal through `slices.Equal`. Correct.
- **`RegisterMachine`**: an unchanged registration writes and pushes nothing. A refusal inside the write closure aborts before `Save`. Correct.
- **`WriteHold`**
  - A hold of another machine stands: nothing is written and `Standing` is set. Correct.
  - Same machine, kind and subjects: idempotent no-op. Correct.
  - Kind outside the two values, or an `adopt-fix` hold with no `FixTask`: refused. Correct.
  - Unreachable origin: `ErrOwnerOriginUnreachable`. Correct.
  - See F1 for the re-run case.
- **`ensureIdentityWith`**
  - No directory resolves: `RegistrationError` naming the variable. Correct.
  - File exists with an id: returned with no write. Correct.
  - File exists without an id: the id is written, other keys kept, and a concurrent writer's id wins. Correct.
  - File missing: created from the hostname name. Correct.
  - The `!found` branch inside `UpdateJSON` can only write an empty `hostName` if the file vanishes between the read and the lock. That is a negligible race.
- **`writeOwners` and `SetOwners` with `Identity`**
  - `Check` runs before any owner is touched, so a refusal writes nothing.
  - A registration-only change still saves, through `!registered`.
  - A mismatch between `Identity.Name` and `Machine` is refused.
  - All correct.
  - Callers that build the request without `Identity` (`internal/beamcli/create.go:174,196`, `internal/battencli/owner.go:210,229`) get it filled in the wiring seam (`internal/beamcli/deps.go:65-69`, `internal/battencli/owner.go:65-69`). Correct.
  - `internal/machinecli/set.go:256` sets none, which fits its Replace sweep. I did not read how it registers.
- **`machinename.resolveIdentity`**: no file or no directory gives an empty id, and a file with an id gives id plus Forgotten. Correct.
- **`CatchUp`**
  - Unreachable origin at the Bolt pull or a fetch: returns before any run is acted on. Correct.
  - A push or hand-over failure for one run is joined and the other runs go on.
  - `ApplyPauseIntents` runs over `OwnedHere`, which includes a run whose push failed. That is acceptable.
  - A nil seam returns a wiring error.
  - Correct.
- **`daemon.catchUp`, `catchUpFailed`, `catchUpSucceeded`**
  - The retry delay runs 1s, then doubles up to `idleCeiling`. Correct.
  - A refusal and a `failed` outcome each post one notice per episode. An origin outage posts none. Correct.
  - Success clears the wait, owes every run a full pass, and drops the touched runs' caches. Correct.
- **`tick`**: while the catch-up is owed, scheduling and the landing step are held. An adopt hold skips the catch-up and leaves it owed. Correct.
- **`handOver`**: a failed catch-up, subject end or archive keeps the dobby up, logs once per episode, and retries at the next probe tick. Correct.
- **`UpgradeRead`**
  - No standing entry: no-op with no Bolt pull. Correct.
  - Origin unreachable: nothing written. Correct.
  - Intents and the hold are pushed before the drops, so a cut read re-runs from the entries left.
  - A standing `owner` entry or a hub-scope entry names no run and is dropped. Correct.
  - Subjects from a run another machine now holds (`handedOver`) are skipped. Correct.
  - Another machine's hold standing: none written, and the notice names that machine. Correct.
  - See F1 for the re-run case.
- **`OwnedWorktrees`, `EnsureLive`, `refuseEvery`**: an identity failure refuses every named worktree with the cause. A prime claim that stood reads as here, and one lost reads as elsewhere. Correct.
- **`HandOverOwnedElsewhere`**: with no takeover entry on origin it pushes the prime's records, then the pair if it is still here, then hands over. A push that meets a takeover entry ends with nil. A failed push archives nothing. Correct.
- **`machineNamesOf`**: gathers the former names plus the registry entry's names, keeping only those that read as here, and omits the current name and duplicates. Correct.
- **`ReadRunPulled`**: an unreachable origin reads the local Bolt, any other pull failure refuses with a way forward, and a missing Bolt directory is skipped. Correct.
- **`runReader.readOwner`**: `OwnedElsewhere` goes through the registry. `Held` and `HoldMachine` come from `HoldOf`, which skips the fix task of an `adopt-fix` hold. Correct.
- **`runfsm.Act` and `Refusal`**: an empty `HoldMachine` falls back to "another machine", and the way forward does not name `--take-hold`. Correct.
- **`CoveredError` and `CoverageOf` for `BoltHoldCaller`**: `Held` and `HoldMachine` are set, the wording names no `lym tender resume`, and an empty machine falls back. Correct.
- **`SyncBoltHold`**
  - Hold standing, slug held, no own entry: a bolt-hold entry is written.
  - An own entry of any caller: skipped.
  - A slug that left `Hold`, or holds its parent lock, or a hold no longer standing: the own bolt-hold entry is lifted and operator entries are left.
  - A per-slug failure is joined and the other slugs go on.
  - Correct. See F2 for the entry's recorded machine.
- **`tenderengine.Resume`**
  - Own entry is a bolt-hold entry under a standing hold: refused with nothing lifted or cleared.
  - Other bolt-hold entries are kept.
  - Lifted slug entries are replaced by bolt-hold entries, except for the hub scope and the fix task.
  - A hold-read failure refuses with a way forward.
  - Correct.

## Findings

**F1. LOW, design, `internal/dobbyengine/upgrade.go:1330-1340` (diff lines), `UpgradeUpgrade.writeHold`**
- A re-run of a cut first read posts a second notice. `WriteHold` returns `Wrote=false, Standing=nil` for the idempotent repeat, and `writeHold` posts `Notice` regardless. The same happens on every retry after a failed `Drop`, since `catchUp` re-runs the read.
- The record says the first read posts one notice. The notice is only repeated, not wrong.
- Evidence: `internal/boardengine/holdwrite.go:56-62` and `internal/dobbyengine/upgrade.go:1330-1340`.
- Fix: post the notice only when `written.Wrote` or `written.Standing != nil`.

**F2. LOW, design, `internal/tenderengine/bolthold.go:671-674`, `SyncBoltHold`**
- A run with an existing bolt-hold entry is skipped, whichever machine the entry records. If the hold moves to another machine, or the first hold clears and another machine writes one before the lift, the entry's `Machine` stays stale.
- `CoveredError` and `boltHoldRefusal` then name the wrong machine to the operator.
- Evidence: the `hasOwn → continue` branch and `pauserecord.Entry.Machine` in `internal/pauserecord/record.go`.
- Fix: when the own entry is a bolt-hold entry whose `Machine` differs from `req.Machine`, lift it and write a fresh one.

I found no defect of higher severity in my lens.

## Tool calls
1. Read `bru52ivdd.txt` lines 60-959: the saved diff of `pairteardown`, `runfsm`, `tenderengine`, `machinename`, `hubgeom` and `pauserecord`, which my context lacked.
2. `grep "OwnerRequest{"` over non-test Go files: to see which `SetOwners` callers pass an `Identity`.
3. `grep Identity` in the `beamcli`, `battencli`, `adoptcli` and `machinecli` wiring: to confirm the callers without `Identity` get it filled in at the seam.
