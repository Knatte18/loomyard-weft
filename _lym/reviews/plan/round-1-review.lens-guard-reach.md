```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: for each guard, refusal, branch or test row the plan adds, I checked whether a real entry point can reach it and whether the site it guards recurs somewhere unguarded.

## Per guard: reachable or not

- **Card 2, `Shed.stepLocked` refuses a cancelled status:** reachable. `lym batten step` on a finished cancel gets through, because card 9's guard refuses only an intent without a finish (`startguard.go:276-287`). For `lym batten run`, card 7's act check refuses it first.
- **Card 2, `Goto` / `Relaunch` refuse a cancelled status:** reachable. Goto on a finished-cancel record passes card 9's guard.
- **Card 4, `Act` branches:** reachable through `TestAct` inputs. In production, `PushOwedOffline`, `CodeTipMissing` and `Held` stay zero, as the shared decision `act-inputs-this-part-fills` allows.
- **Card 4, `WayForward` `lym batten cleanup --apply` for a cancelled run whose pair is gone:** reachable, but it fires on a kept cancel viewed from a machine that lacks the pair. See F1.
- **Card 6, `planPass` returns no pass for `Wait` or `HandOver`:** reachable.
- **Card 6, `drivePasses` runs `ownedElsewhere` for a `HandOver` run:** reachable, but its owner pause hits card 14's new owner refusal. See F2.
- **Card 7, act refusal in `startGuard.refuse` for `run`:** reachable. It runs inside `refuseAtStart`, before `claimStart` and every owner and covered check (`arm.go:347-355`). Inputs that reach it: a cancel intent, or an owner elsewhere on the local Bolt.
- **Card 7, `passExitCode` mapping `hand_over` and covered:** reachable from a hand run. From the dobby only in a race, because card 6 gives those runs no pass. Not a defect.
- **Card 8, act refusal in `lym loom resume`:** reachable through a cancel intent. `hand_over` sits behind the existing owned-elsewhere refusal, so it is reached only when the local Bolt disagrees with origin. The card names the cancel-intent input, so this is not a finding.
- **Card 9, live-cancel refusal through `CancelLock`:** reachable by two concurrent cancels on one machine.
- **Card 9, a failed push while origin answered refuses before anything ends:** reachable.
- **Card 9, offline cancel commits the intent unpushed and names the rerun:** reachable (`judgeOrigin` sets `endOnly`, `cancel.go:531-564`).
- **Card 9, start guard refuses an intent without a finish:** reachable for `step` and `goto`. Not reachable for `run`, because card 7's act check runs first. See F5.
- **Card 9, the cancel-intent check in `refuseStartOwnedElsewhere`:** reachable only when an intent lands between `refuseAtStart` and `claimStart`, as the existing pull-race test does.
- **Card 9, cancel with no batten record (`RecordCancelIntent` returns found=false):** the state can occur; the card prescribes no branch for it. See F4.
- **Card 11, `deleteCancelledBattenRecord` in `Teardown.Run`:** reachable through `lym fabric remove`, since the remover reaches the teardown for an active or absent entry (`fabriccli/remove.go:54-95`). It also runs on local-only removals. See F3.
- **Card 11, cleanup skips a cancelled record whose pair stands:** reachable only on the machine that holds the pair. See F1.
- **Card 11, cleanup removes a cancelled record whose pair is gone:** reachable, and wrongly so on a machine without a kept pair. See F1.
- **Card 12, a failed publish halts the row:** reachable on a push failure.
- **Card 14, owner refusal in `tenderengine.Pause`:** reachable through `TestPause`'s fake. It also catches the internal owner pause. See F2.
- **Card 14, act refusal in `lym tender resume &lt;slug&gt;`:** reachable through a cancel intent. `hand_over` also blocks the hand lift of an owner pause. See F6.
- **Card 14, offline `push_owed`:** reachable.

## Findings

### F1 — BLOCKING, class decision, cards 11 and 4
Cleanup judges "pair stands" from local pair parts only, so a machine without a kept pair deletes that kept cancel's record.
- Card 11 fills `PairStands` from `Topology.PairParts`, which reads locally with no network call (`fabricengine/pairparts.go:16,35`).
- The prime's batten record is tracked and pushed, so on a machine without the pair a kept cancel reads "no pair part left". `lym batten cleanup --apply` then removes the record, commits and pushes, while the pair still stands on its owner and the task branch stays on origin.
- Card 4's `WayForward` sends the operator straight there: `lym batten cleanup --apply` for a cancelled run whose pair is gone.
- This breaks D3, which refuses a record whose pair stands. It also breaks D1 and its acceptance line: a kept pair's record reads `cancelled` on every machine.
- Fix: judge "pair stands" from the task branch on origin as well, the input D1 already uses. `WayForward` should name `lym fabric remove` whenever the cancel was a keep.

### F2 — BLOCKING, class consistency, card 14 (with card 6)
Card 14's new owner refusal in `Pause` also refuses the internal owner pause, whose whole purpose is a run another machine owns.
- `tendercli.PauseOwnedElsewhere` calls `tenderengine.Pause` with a slug and `Caller: pauserecord.OwnerCaller` (`tendercli/deps.go:228-238`).
- Batten's `PauseOwned` seam goes through it (`battencli/wire.go`), and card 6 relies on it: `ownedElsewhere` is run "so the owner pause still lands on this machine".
- Card 14 says `Pause` with a slug asks `RunOwner` first and refuses before writing anything. Only a hub-wide pause is exempted, not an `OwnerCaller` pause.
- So the owner pause is refused. The card would also write a pause intent into a record the run's owner owns, against D3's "only the owner writes a run's records". The existing `TestPauseOwnedElsewhere_PausesTheSlugOnceUnderTheOwnerCaller` (`tendercli/cli_test.go:238`) reaches this.
- Fix: limit the owner refusal and the intent write to the operator's `lym tender pause &lt;slug&gt;` (`pauseCmd`), never the `OwnerCaller` path.
- Not verified within my tool budget: that `PauseOwnedElsewhere` builds its deps through `HubDeps`, which wires `RunOwner`.

### F3 — MEDIUM, class consistency, card 11
The record deletion also runs on local-only removals, so a non-owner machine deletes a tracked record while the pair stands elsewhere.
- `lym fabric remove` sends `Request.Local` for an `owned_elsewhere` or `ended` verdict (`fabriccli/remove.go:85-104`). Such a removal deletes no origin branch.
- Card 11 deletes, commits and pushes the cancelled record after every successful removal in `Teardown.Run`, with no `Local` check.
- On an owned-elsewhere slug, this machine therefore deletes the owner's tracked batten record while the pair and the task branch stand.
- Fix: run the seam only when `Request.Local` is false.

### F4 — MEDIUM, class consistency, card 9
A cancel of a slug with no batten record ends the run with no intent anywhere, and the card does not say this is allowed.
- `RecordCancelIntent` returns found=false and writes nothing when there is no batten record.
- `refuseNothingLeft` still lets the cancel proceed on a pair part, a live session, a claim or live processes (`cancel.go:675-678`).
- Card 9 does not say what follows found=false. The cancel then ends processes with no intent on origin, against D3's "commits and pushes it before it ends anything".
- Fix: the card should state that this case either proceeds as today, bounded, or refuses.

### F5 — LOW, class consistency, cards 9 and 7
Card 9's intent refusal for verb `run` in `startGuard.refuse` cannot be reached.
- Card 7 asks the act rule "first, before every other check" in `startGuard.refuse`. A cancel intent without a finish derives `cancelling`, and the act rule answers `wait`.
- Card 9's new `run` rows in `TestStartGuard_Refuse` therefore reach the intent refusal only with an Act seam that ignores the intent.
- Fix: card 9 should keep its intent rows to `step` and `goto`, or define a nil Act seam as `act` and say so.

### F6 — LOW, class consistency, card 14
The act check on `lym tender resume &lt;slug&gt;` answers `hand_over` for a run another machine owns, so it refuses the hand lift of that slug's owner pause.
- `pauserecord/doc.go:23` says owner and adopt pauses are "lifted only by hand, with `lym tender resume` or `lym batten cancel`".
- The way forward (`lym adopt` here first) exists, but no card updates that contract.

### F7 — LOW, class consistency, card 11
For a cancelled record, `BattenFollowUp` falls through to "run `lym batten cancel &lt;slug&gt; --remove`" (`pairteardown/followup.go:33-40`).
- Card 11 does not place its delete before `Teardown.Run`'s follow-up read, and does not teach `BattenFollowUp` the `cancelled` state.
- So a cancelled record that outlives a fabric remove, for example after a failed commit, is followed by advice to re-cancel an already finished cancel, not to retry the removal.

## Tool calls
1. `battencli/startguard.go:276-300` and `drive.go:115-150`: guard order inside `refuse` and the pass-exit mapping; context held only the struct and callers.
2. `battencli/startguard.go:480-530` and `owner.go:260-285`: where `refuseAtStart` and the owner check run; not in context.
3. `battencli/arm.go:325-375,620-648`: arm order relative to the covered refusal; not in context.
4. `pairteardown/followup.go`: `BattenFollowUp` and `followUpOver`, which `Teardown.Run` calls after a removal; not in context.
5. `fabriccli/remove.go:20-140`: whether `lym fabric remove` reaches `Teardown.Run`, and its `Local` verdicts; not in context.
6. grep over `tendercli`, `pauserecord` and `tenderengine/resume.go` for the owner-pause caller and its lift; the internal `Pause` caller was not in context.
