```yaml
lens: fix-proposal
outcome: reported
```

no findings

The wording matches the code.
- `internal/loomcli/landingdeps.go:102` sets `EscalateJudgedFirst` to true for `RecipeLoom` and `RecipeDarn` only, so "loom or darn escalate" is the exact exception.
- The `fixerrun.go:312` branch is the only place the prefix changes. A pawl escalate and every other end keep `LandingStopPrefix`.
- The line 420 spent call passes the flag too, but `fixerEndSpent` never reaches that branch.
- The wording agrees with `doc.go:122`, `parentalign.go:122` and `finalize.go:593`.
- The next doc sentence ("An escalate end with escalateJudgedFirst true opens with EscalateStopPrefix instead…") now partly repeats the opening sentence. That is a redundancy the review's own fix key produced, not a defect, so I rate it below `LOW`.

Tool calls:
1. Grepped `haltReason(` callers, because my context lacked the call sites and the flag values they pass.
2. Grepped `EscalateJudgedFirst`, because my context lacked where the flag is set and which recipes it covers.
