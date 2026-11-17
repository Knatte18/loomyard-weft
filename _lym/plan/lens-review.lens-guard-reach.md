```yaml
lens: guard-reach
outcome: reported
```

Task: a guard-reach review of the plan's cards. For each guard, refusal and test row, I checked whether a real entry point can reach the state it guards.

**BLOCKING**

1. **Card 5, `UnderOwnHold`, the guard on the dobby hand-over and the orch watcher: it guards the wrong state.**
   - `UnderAdopt` is asked only while `elsewhere` is true. See `internal/dobbyengine/handover.go:41` and `internal/orchengine/owner.go:57`.
   - Card 2's `WriteAdopt` writes this machine as owner of every subject at step 1, before any other step. So on the adopting machine, no subject worktree ever reads as owned elsewhere while the hold stands. The state the old record guarded, a subject owned elsewhere while the adopt runs, cannot occur any more.
   - The only worktrees that now reach `elsewhere && UnderOwnHold` are beams outside the adopt's subjects that another machine owns. Examples are a nested beam under an adopted root that keeps its own owner, or any unrelated foreign-owned beam.
   - `Owners.HoldOf` holds every slug (`internal/boardengine/registry.go:157-167`). So those dobbies stay idle and never hand over, and `ownerArchiveBeforeRefusal` skips their archive. This lasts for the hold's whole life, which includes every `adopt-fix` round.
   - The old `Watch` did not cover them: `ownerSubject` resolves a beam with its own owner to itself, which the record does not list (`internal/adoptrecord/watch.go:97`).
   - No row in `TestUnderOwnHold` tries a beam outside the hold's subjects.

**MEDIUM**

2. **Card 9, `movedRun`, the guard that keeps the fix task untouched on a re-run: it cannot be reached in the real state.**
   - A run is excluded only when the subject's `PriorOwner` in the hold reads as this machine.
   - On a real adopt from another machine, card 2 sets `PriorOwner` to the old owner, and "the hold keeps `PriorOwner` across a re-run".
   - So a plain re-run under this machine's own `adopt-fix` hold judges the fix task's run as moved. `takeOverRuns` then archives it and resets its branches through `DropLocal`, which breaks the criterion "leaves the fix task's branches and processes untouched".
   - Every run already taken over is re-reset the same way, because its origin takeover entry names this machine, not another.
   - The row `AReRunUnderItsOwnAdoptFixHoldKeepsKindTaskTriesAndFilesNoSecondTask` can pass only over a fake whose `PriorOwner` is this machine, which is not the state a real re-run is in.
3. **Card 2, row `TwoClonesAdoptingTheSameBeamEndWithOneOwnerAndTheLoserRefusesNamingTheWinner`: it cannot reach the rejected-push path.**
   - `WriteAdopt` runs through `commitBoardWriteAndPush`, which takes no hook between the commit and the push (`internal/boardengine/registrysetter.go:166-188`). Only `OwnerRequest.BeforePush` has such a hook (`internal/boardengine/ownerset.go:177`).
   - `AdoptRequest` declares no hook, so the test cannot push the other clone's adopt between this clone's pull and its push.
   - A push before the pull reaches refusal case 4 instead. Card 1's whole-commit refusal and card 2's post-push re-read refusal stay unreached by any tier 1 row.
4. **Card 3, row `ATakeoverPushRejectedOverTheOldOwnersLaterTransitionWritesOntoOriginsTipAndLands`: it cannot reach the retry.**
   - `TakeOverRun` fetches at step 1 and pushes at step 4 inside one call. `TakeOverRequest` names no seam or package hook for planting origin's commit "after the fetch". The precedent for such a hook is `owedTipPushedHook` (`internal/fabricengine/handover.go:28`).
   - The D2 criterion this row carries is therefore unreached.
5. **Card 9, `holdStands` "before each step from here on": it would stop the green path.**
   - On green, `HoldCheck` clears the hold (card 8, step 3).
   - `runBeamSteps` comes after it as a separate step. If `holdStands` runs before that step, it finds the hold gone, stops and starts no dobby.
   - The card does not exempt the bring-up after a green check. Its row `AGreenCheckBringsEachSubjectUpAndReturnsOk` contradicts the guard as written.

**LOW**

6. **Card 5, row `AnAdoptFixHoldDoesNotCoverItsFixTask`: no real caller reaches it.**
   - The accessor is asked only with a dobby's or orch's worktree slug: the prime `""` or a beam (`internal/beamcli/dobbydeps.go:294`, `internal/orchcli/watch.go:105-108`).
   - A fix task is a task, never such a worktree, so the exclusion is never reached through a real entry point.
7. **Card 8, `DropBoltHold` on green: the site it guards is written again by a path the card does not guard.**
   - A dobby already running on the adopting machine writes bolt-hold entries from the local Bolt it read in its tick (`internal/beamcli/dobbydeps.go:116-139`).
   - If that tick read the hold before `ClearHold` and writes after `DropBoltHold`, an entry stands when the adopt returns. That breaks the criterion "a green adopt check leaves no bolt-hold pause entry on the adopting machine when the adopt returns".
   - The card neither serialises this nor re-drops after the bring-up.

Reachable, no finding:
- Card 1: the replay refusal and registry `Lost`.
- Card 2: refusal cases 1–5, the registration refusal, unreachable origin, and the `FixHold`/`ClearHold` refusals.
- Card 3: the `DropLocal` anchor stop and the mark drop.
- Card 4: the pattern rows.
- Card 8: no own hold, subject outside the hold, lock held, open own fix task, and limit reached.
- Card 9: `--take-hold` with `--all`, `--take-hold` with `--archives`, the `refuseStandingMove` gate, another machine's hold, other-set refusal, and the `--here` outcomes.

Tool calls:
- grep of `replay_test.go` and `droplocal_integration_test.go`, to confirm that `TestFabricReplay` and `TestDropLocal` exist and which package each is in; my context did not hold the test files.
