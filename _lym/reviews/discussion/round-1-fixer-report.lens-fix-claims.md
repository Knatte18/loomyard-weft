```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked every claim the fix added or rewrote against the code at HEAD.

No findings.

What each claim was checked against:

- **D4 Reason or Error claim (F1):** `stopFromStatus` (`driverules.go`) sets `Reason` only for a paused status and carries the halt text in `Error`, and `budgetExhausted` reads both fields. The claim holds.
- **D4 Bound claim on the shared budget (F3):** a fixer escalate returns its fixer attempt (`s.rec.Spent--` in `fixerrun.go`). `Settle` (`shedarbiter/call.go`) counts the spawn budget per run from the attempt records and refuses with "spawn budget spent" once it is used up, whatever produced the stop. Each new escalate stop has a new stop key because the history grows, so each one spawns a fresh arbiter. The refusal goes through the hand-on path D4 already names. The claim holds.
- **D3 claim on the decision halt prefix (F4):** `arbiterstop.HandsBack` matches `shedadapters.DecisionHaltPrefix` as a prefix, as shown in my context. The claim holds.
- **D3 pawl hand-back claim and the pawl acceptance criterion (F2):**
  - `haltReason` opens every reason, escalate included, with `LandingStopPrefix`.
  - `landingshed.HandsBack` (`handback.go`) is true for that prefix at Parent-Align and Finalize.
  - A pawl escalate that keeps `LandingStopPrefix` therefore hands back.
  - Pawl has a Parent-Align (the `codeReviewOff` comment in `landingdeps.go`), so the criterion naming both rows can be reached.

  The claim holds.
- **Constraint "pawl's nil `RerouteReviewed` still halts a spent or over-cap fixer":** the comment and the `recipe != shedrun.RecipePawl` guard in `landingdeps.go` back it. The claim holds.

Tool calls:
- I read `fixerrun.go:250-260`, `shedarbiter/call.go:105-125,180-195` and the `HandsBack` body in `landingshed/handback.go`, because my context had only the review's line citations for the attempt give-back and the spawn cap, not the code, and had not shown the body of `landingshed.HandsBack`.
