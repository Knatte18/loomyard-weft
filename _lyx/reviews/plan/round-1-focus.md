---
round: 1
focus:
  - "Card 3's bound claims every `restampFingerprint` caller runs only after a foreign-edit check passed in the same call. Check that claim against the code for each site the card lists (begin-batch, `Run`'s `restampAndSaveFingerprint`, record-batch, recovery's binding and drift-repair restamps, validate's `RestampPlanBaseline`). The Shared Decision says a rewrite is re-baselined even when the call fails, so settle whether any site can restamp, and so move begun `CardHashes`, while an unadopted foreign edit is on disk."
  - "Card 9 adds the refusal-spec row 'integration regression not fixed', and the escalation path has no row today. The Refusal Way-Forward Invariant wants the row plus a reaching test in the same commit. Settle whether card 9's tests assert that the stuck reason names the way forward the row states (pre-fix head, fix commits, `git reset --hard`, verify and commit), or only that the run escalates."
  - "Card 9 steps 1–2: step 1 records `IntegrationFix` (spending the attempt) before the nil-`FixStarter` wiring guard and before a start error at step 2. Settle whether `IntegrationFix.Result` gets set on those paths, and whether a wiring error then reads as `spent` on resume. D5 says the attempt is recorded when spawned, so check that the card's ordering and failure paths agree with that."
  - "Card 7 checks the plan directory only 'when it lies inside' the worktree. Under the attack-surface rule, settle whether the card states what the check lets through when `planDir` lies outside the worktree, and whether the `_lyx` check alone bounds that case."
  - "Card 8 edits `contracts/stencils/landing/landing-template-describe.md` so the Describe stencil carries the new summary section. Under answer-key fidelity, settle whether D5 (summary.md naming fix commits 'for the review rows below to judge') licenses this, or whether it is scope the decision record does not name."
---

The plan covers D1–D6 and the acceptance criteria card by card, and each new guard or routing edge states a bound.
I left out `ImpactSummary` presence, handle declarations and verify-section shape because the gate checks them.
I did not promote verify coverage: the section runs every targeted package plus `go test ./...`, and both integration-tagged packages are run with `-tags integration`.
