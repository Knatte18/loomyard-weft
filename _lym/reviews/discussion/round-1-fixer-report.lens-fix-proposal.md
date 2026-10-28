```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out as proposed (F1, F3 and F4, plus F2 with option (a)).

no findings

- F1: I found no case that breaks the proposal. `stopFromStatus` passes `st.Error` to its hands-back check, so a blocked status-built stop does carry its halt text in `Error`. A paused stop's `Reason` holds a pause reason, which can never open with the escalate prefix.
- F2(a): I found no case that breaks the proposal. A pawl escalate that keeps `LandingStopPrefix` goes through the existing hand-back. That avoids the refused `lym loom decision add` and the resume loop, and it matches the Constraints' "Pawl's landing keeps its own treatment".
- F3: the claim holds. The spawn cap counts every attempt record of the run (`internal/shedarbiter/call.go:116-117`, `188-189`). An escalate gives its turn back (`fixerrun.go:89`). Once the cap is spent, the arbiter refuses, and that stop goes back through the hand-on path.
- F4: the proposal is sound. `DecisionHaltPrefix` is `"decision halt: "` and `LandingStopPrefix` is `"landing stop: "`, so any new prefix that avoids both is clear of the checks in `arbiterstop.HandsBack`.

Calls:
- Read `internal/shedarbiter/call.go` around lines 110-120 and 183-192 to check F3's shared-budget claim, since my context did not show the arbiter's spawn counting.
- Grepped the `DecisionHaltPrefix` and `LandingStopPrefix` constants to check F4, since my context did not show their values.
