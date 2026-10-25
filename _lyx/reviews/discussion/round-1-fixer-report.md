# Fixer report — discussion, round 1

Every finding's premise was checked against the code at HEAD and held.
All 17 findings are fixed in `_lyx/discussion/decision-record.md`, with a Round 1 entry and ledger questions Q17–Q20 in `_lyx/discussion/support-log.md`.

## Changes

- F1 (BLOCKING): D4 now says the owner check is due only after a sleep the same process watched, a pause lift or a probe while owned elsewhere (`foldOwnerTrigger`, `innerrun_owner.go`).
  Dobby holds the sleep watch and owes every run a full pass after a wake.
  The false clause and D3's owner-check claim are gone; D3 names the full pass and the persisted triggers.
  New criterion: the first pass after a machine wake runs the origin reconcile.
- F2 (BLOCKING): in D5, `settleClaim`'s `claimClear` arm writes `draft` through `writeBoardClaim`, and D5's bound now says no path but an explicit release leaves a task statusless.
  The cancel criterion covers a pair `lyx fabric remove` removed.
- F3 (BLOCKING): D4 inventories the memory record from `innerrun.go`, `notice.go`, `innerrun_binaryrestep.go` and `innerrun_parentlock.go`.
  It lists the producer fields and the `childWait` fields that now persist, each with the rule it keeps; the idle sleep watch is replaced by dobby's recorded sleeps.
  The Open-risks bullet is removed, and the Constraint now separates file markers from in-memory latches.
- F4 (MEDIUM), fixed differently: `holdSpawnWhileCovered` returns `Yield`.
  I did not apply the proposal to make the driver's `PauseHold` yield, because `holdPause` is called only inside stop handling (`internal/sheddriver/drive.go:251,315,418,509`).
  Ending the pass there would leave the stop with no settler, since dobby passes only `running` runs.
  D4 names that hold and Land's land-lock poll as the waits bounded only by what they wait on, with every other in-pass wait's bound stated.
  D12 says how the lift reaches a pass in flight.
- F5 (MEDIUM): in D1, `EndBeforeSessionDown`, reached from `internal/reedcli/up.go`, spares a `RoleDobby` record, so D2's always-a-dobby claim holds; criterion added.
- F6 (MEDIUM): in D4, the pass refuses `--continue-pass` unless the run is `running` at Run-Shed and named, with no full pass owed, in its dobby's state record.
  The flag never skips the owner triggers. Criterion added.
- F7 (MEDIUM): D4 says who resumes: a hand pass, run by the operator or by the orch on the operator's word or after judging the stop, never by dobby.
  The Goal and D6 are reworded to match, and the continue-pass sentence now names every hand pass as a full pass.
- F8 (LOW), fixed differently: the respawn count resets only on a write of the child's status file since the last respawn.
  The proposed reset on a live strand or held lock was replaced after the self-check (proposal lens): a driver that lives a few minutes after each respawn would reset the cap forever.
- F9 (LOW): the rule now sits on `battenAlive` itself, so `driverChecksSkipped` takes it too. The self-check narrowed the dobby arm to `running` runs with no cancel marker.
- F10 (LOW): D8 says only hub-layer values carry over; worktree, local and `run_settings.loom` values apply to loom alone.
- F11 (LOW): D10 counts from the batten history of every run the dobby drives or has driven, on each tick, keyed by run, with a first-start baseline.
- F12 (LOW): D5 states the older-binary case as an accepted bound.
- F13 (LOW): D4 says a re-exec is not a start, and an owed full pass survives it.
- F14 (LOW): D7 uses D4's and D5's keys and says when each is cleared.
- F15 (LOW): D13 defines in flight for the backoff, apart from `max_runs`.
- F16 (LOW): in D4, a batten status write no later than the slug's last pass exit triggers nothing; criterion added.
- F17 (LOW): D11's last sentence is deleted.

## Self-check

- fix-consistency, MEDIUM, the driver pause-hold exit contradicts the yield bound and causes full passes under a pause: fixed differently (merged with fix-proposal 1 and fix-way-forward 1); the driver's pause hold stays in the pass.
- fix-consistency, LOW, D10 dropped "or has driven": fixed as proposed.
- fix-consistency, LOW, D6's stencil has no resume wording: fixed as proposed; D6 states the stencil's resume rule.
- fix-claims, MEDIUM, the land-lock poll ignores a tender pause, so the bound and D12 were false: fixed as proposed; D4 says it ends only at batten's own pause flag, and D12 no longer calls it bounded.
- fix-claims, LOW, `revival`'s failed-revive mark and `reason` are memory, not fresh readings: fixed as proposed; both added to the inventory.
- fix-proposal, MEDIUM, the driver pause-hold exit strands a stop: fixed differently (merged above).
- fix-proposal, MEDIUM, the dobby arm of `battenAlive` makes a halted run unsettled: fixed as proposed; the arm counts only `running` runs with no cancel marker.
- fix-proposal, MEDIUM, a respawn-cap reset on liveness never caps a slow-dying driver: fixed as proposed; the reset needs a status write.
- fix-proposal, LOW, old run records count as landings on a first start: fixed as proposed, with a first-start baseline.
- fix-way-forward, MEDIUM, full passes every probe tick under a pause: fixed differently (merged above); no pass exits at a stop under a pause any more.
- fix-way-forward, MEDIUM, no clearing rule for an owed full pass: fixed as proposed; a start owes one too, cleared only when a full pass exits 0.
- fix-way-forward, LOW, the refusal key is never cleared for an unclaimed task: fixed as proposed; it also clears on a change of the board entry.
- fix-way-forward, LOW, dobby has no handling for the `--continue-pass` refusal: fixed as proposed; the refusal has its own exit code, which dobby reads as a full pass owed.
- fix-way-forward, LOW, a hand pass under a hold exits with no reason: fixed as proposed; the yield prints the hold reason.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
