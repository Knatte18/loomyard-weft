```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked whether the factual claims the fix added or rewrote hold against the code at HEAD.

Claims checked that hold:
- The prime's pair push is records-only: `internal/fabricengine/pushanchored.go:3`, and `internal/fabricengine/failedpush.go:250` names `git push` as the way to push the prime's code branch.
- `adopterFirstRepair` reads whether origin's parent holds the run's work, through `OriginParentHoldsWork`: `internal/landingshed/finalize_owner.go:183`.
- `lym adopt root` for the prime, and a run's subject is its beam or its slug: `internal/landingshed/finalize_owner.go:141`, `:148`.
- `shedrun.ScratchDir` exists: `internal/shedrun/paths.go:201`.
- Removal goes through `internal/pairteardown`: `internal/pairteardown/teardown.go:273`, `:275`, and `internal/pairteardown/doc.go:172` covers the hand-over's removal.
- Finalize hands a run over through `Deps.HandOver`: `internal/landingshed/finalize_owner.go:108`.

Findings:

1. MEDIUM. D5's new hand-over paragraph: "The local parent keeps the run's landing commit, which origin lacks and no landing entry records, so the prime's pull and the next local landing read it as any other ahead commit, with today's Stuck reason…"
   Summary: today's hand-over already tags the run's landing commit and resets the parent when only that commit puts it ahead. The claim that the local parent keeps the commit is therefore false in the common case, and the manual drop it names is a step with nothing left to drop.
   Evidence: `internal/fabricengine/handover.go:4` says it "resets a parent branch the run's own merge put ahead of its upstream", and `:64-65` say "A parent branch that only the run's own merge puts ahead of its upstream is tagged the same way. That branch is then reset to the upstream." The claim holds only when other local commits, such as a newer landing entry's merge, sit on top.

2. MEDIUM. D5's new hand-over paragraph: "its push of the pair's records stops short of the held records commit."
   Summary: the hand-over makes no push of the pair's records branch. What it does push is the local records tip under an owner archive tag, so the held commit reaches origin that way, and the fix does not address that.
   Evidence: `internal/fabricengine/handover.go:70`: "It pushes no branch but the owed code tip, and no tag but the owner archive tags". `:62-63`: "archives the pair's local code and records tips that origin lacks … Each goes under an owner archive tag pushed to origin."

Tool calls:
- grep of removal and hand-over call sites across `internal/`. My context lacked whether every pair removal and the hand-over go through `internal/pairteardown`.
- grep of `internal/fabricengine/handover.go` doc and pushes, and of the hand-over lines in `internal/pairteardown/doc.go`. My context lacked what the hand-over pushes and whether it resets the parent.
