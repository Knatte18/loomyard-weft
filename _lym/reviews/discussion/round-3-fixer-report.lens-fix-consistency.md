```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: I checked that the round-3 change set agrees with the sibling decisions, criteria, constraints and cross-references it touches or depends on.

- **MEDIUM**, D5's diverged-refusal case where "origin's parent already holds it under another commit". The fix makes the run's landing entry go with no `done` write. It does not say what happens to that entry's pair hold or the held records commit. The hand-over paragraph next to it does: it "drops the run's landing entry and its pair hold before it removes the pair, and its push of the pair's records stops short of the held records commit". In the drop case, the entry's going lifts the hold that D5 (:145-146) and D1 (:60) give it. So the pair's pusher would then push the held records commit, and with it the run's ended entry, to origin. Origin's records already belong to the new owner, who re-ran Finalize there. The twin rule was written for the hand-over and left out of the drop case. The drop case should either release the hold through the hand-over or state that the held commit is never pushed.

- **LOW**, D3's rewritten sentence "Clearing that record through its way forward ends the refusal: a `lym fabric push` that lands for a pair's branch, a beam parent's included, and a `git push` in the prime's worktree that lands for the prime's code branch (D5)". The old text was an example ("such as"). The new one is a list that leaves out the prime's records branch, whose refusal D3 :95 counts against the prime's target. A reader can no longer find that branch's way forward here.

- **LOW**, D3 :90 still says the parent target's entry stays "so the all-clear waits for every landing's `done` write". After the fix, two landing entries go with no `done` write: the diverged drop case, and the hand-over that drops the run's landing entry. That reason clause is now stale.

- **LOW**, two new refusals point at "(D7)", but D7 :209-210 doesn't list either one. One is a run owned elsewhere. The other is "a parent push whose tip carries it [an unrecorded landing commit] is refused (D7), naming that commit". D7 defines a refusal as a rejected push that stays rejected, a login failure or a moved branch. Neither new case is a git rejection, so the cross-reference names a decision that does not cover them.

Further calls:
- I read `internal/shedrun/paths.go:195-205`, the `ScratchDir` definition, to check that D9's new base file is named through `internal/lymdirs` as PATTERN-lym-files requires. It is, so there is no finding. My context had only the grep line for that function, not its body.
