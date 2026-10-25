```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the F2 fix added or rewrote holds against the code at HEAD.

Findings:

1. LOW. Changed text: D4, "A first pass with no status file that `refuseStartOwnedElsewhere` refuses exits with the same code and stays a refused start under D5."
   Summary: this sentence says every refusal by `refuseStartOwnedElsewhere` gets the owned-elsewhere code, but some of that function's refusals have nothing to do with the owner.
   Evidence: `internal/battencli/owner.go:106-135` can also refuse through `thisMachine` (the hostname can't be read), `startRefusal` (a pull or prime-owner claim fails, for example with an unreachable origin), `refuseCancelMarker`, and a failed owner read or resolve.
   Only the `ownedElsewhereRefusal` return at `owner.go:133` names another machine as owner.
   The sentence before it scopes the code to refusals that read another machine as owner, so this is a wording gap, not a contradiction.
   Rewording it as "a first pass with no status file refused as owned elsewhere by `refuseStartOwnedElsewhere`" would close it.

Claims I checked that hold:
- `resumeRefusal`'s `EntryOwnedElsewhere` arm returns `ownedElsewhereRefusal` (`resumereconcile.go:81`).
- `resumeOwnerClaim`'s lost claim returns `ownedElsewhereRefusal` (`owner.go:251`).
- `refuseResumeUnreachable` builds its own owned-elsewhere error (`owner.go:279`), so naming its owned arm separately from `ownedElsewhereRefusal` is right.
- `ownerPreflight` reaches `refuseStartOwnedElsewhere` for a found status at Worktree-Create when `taskWorktreePresent` is false (`owner.go:203-209`), and for a missing status file (`owner.go:200`, `arm.go:395`).
- `claimBoardEntry` runs inside the Worktree-Create row's `CreateWorktree` before any pair exists (`wire.go:809-813`), so a lost claim fails that row and leaves no pair.
  The next pass therefore sees a Worktree-Create status with no pair (`taskWorktreePresent`, `wire.go:244`) and reaches `refuseStartOwnedElsewhere`.
  This depends on a failed row leaving `CurrentProducer` at that row; `persistMarked` sets it from the driver's next producer (`shedengine/run.go:864`).
  Within my call budget I did not trace which producer a row failure passes in.

Tool calls:
- grep for `CurrentProducer` assignments and the body of `taskWorktreePresent`: I needed to confirm the producer stays put on a row failure and how pair presence is judged, and my context had neither.
- `sed -n 845,870p internal/shedengine/run.go`: to see how `persistMarked` writes `CurrentProducer`, which my context lacked.
