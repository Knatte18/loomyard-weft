```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each proposal the fix carried out (F2, F3, F4, F5, and F1's beam-start entry point).

Findings:

1. **LOW.** Changed text: the D5 acceptance criterion rewritten for F4, "…except a slug already holding an entry of another caller, which keeps it and gets none; an `owner` entry is turned into one in place, keeping its pause time."
   - Summary: an `OwnerCaller` entry is itself "an entry of another caller". With the exception placed first, a test written to this criterion would leave an `owner` entry standing and give it no `adopt` entry. That contradicts the conversion in the same bullet.
   - Evidence: D5 lists the conversion ("An existing `OwnerCaller` entry … is turned into an `AdoptCaller` entry in place") before "A slug already holding an entry of another caller keeps it and gets none", so D5 makes the conversion the exception. The proposed criterion reverses that order and never says "other than `owner`".
   - Concrete case: a moved run holding an `owner` entry. Read literally, the criterion expects it to keep the `owner` entry and get no `adopt` entry.

2. **LOW.** Changed text: D5's new sentence for F2, "The step turns the existing own-scope `OwnerCaller` entry of each adopted beam's own pair slug into an `AdoptCaller` entry the same way", and the new D5 criterion.
   - Summary: D5 says the dobby gives an adopt-paused slug "no pass of any kind and no task start until it is lifted". Applied to a beam's own pair slug, that may halt work across the whole beam, not just the beam pair's own sessions. The proposal does not say which.
   - Evidence: D5: "The dobby treats it as it treats the hold today: no pass of any kind and no task start until it is lifted, then the full pass at the lift." D7's Bound: "every adopted run is paused (D5), so the dobby starts no adopted work". The beam slug is not a run, so it is unclear whether the dobby's adopt-pause treatment reaches the beam's runs.
   - Concrete case: `lym s` in a beam whose earlier `lym beam start` wrote an own-scope `owner` entry. After the adopt, that entry is an `adopt` pause. Whether the beam's dobby then starts nothing in the beam until `lym tender resume <beam>` is left undecided. Before this change, today's settled step lifted that entry automatically.

No case against F3 or F5. F3: a resume from `reconcile` re-runs `reconcile` and `pairs` and replaces their stored outcomes, and the done steps after them keep theirs. F5: a worker that loses the lock to a discard exits, and its caller then finds no record from a later start and exits non-zero, as D1 says.

F1, outside this lens (fix-claims): the new D11 Bound says "that adopt's reconcile meets the tips through its local-commit stop". That holds only when origin has moved. A tip that is only ahead of an unchanged origin may not trigger a local-commit stop. I did not verify this.

Tool calls:
- grep of `internal/fabricengine` for the local-commit stop condition, to test F1's new Bound claim. The context did not hold the reconcile code, and the result did not settle the claim.
