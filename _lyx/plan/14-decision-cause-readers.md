# Card 14 — decision-cause-readers

**Edit:**
- `internal/arbiterstop/doc.go`
- `internal/arbiterstop#Stop`
- `internal/arbiterstop#Eligibility`
- `internal/arbiterstop#Check`
- `internal/arbiterstop#TestEscalationEligible`
- `internal/arbiterstop#TestCheck_RefusesLandingHalts`
- `internal/battencli#battenCLI.wire`
- `internal/loomcli/circling.go`

**Uses:**
- `plan:internal/shedadapters#DecisionHaltPrefix`

**Intent:** D8: the arbiter refuses a `decision` escalation and hands it back to the parent, a decision halt stays the parent's, and the `circling` verbs name the new cause.

- Awaiting stops: `Eligibility` already refuses every awaiting cause but `hard-cap` and `circling`, so a `decision` escalation reaches its default refusal, which ends with `HandBackWayForward`; the arbiter can record a decision with `--by arbiter`, and leaving it eligible would let an agent settle what only the operator may.
  The package doc names `decision` beside `budget` among the causes that go to the parent.
  `TestEscalationEligible` gains a pending `decision` row wanting false, and its refused rows also run `Check` over an awaiting status and assert the error ends with `HandBackWayForward`, the refusal's way forward reached by a test.
- Blocked stops: card 12's hard-cap halt over a held refusal is a blocked stop at Discussion-Bouncer whose Reason opens with `shedadapters.DecisionHaltPrefix`, and `Eligibility` admits every blocked stop but a landing halt, so the arbiter, the Go driver's escalation and batten's binary re-step would resume it themselves and spend a judge spawn each time.
  `Stop` gains `DecisionHalt`, which `Check` sets for a blocked status whose error opens with the prefix, and `Eligibility` refuses it, saying the halt at the producer is a decision halt only the run's parent lifts, ending with `HandBackWayForward`.
  `battenCLI.wire`'s `HandBack` seam also marks a blocked status whose error opens with the prefix, so the binary re-step leaves it to the orch, whose resume is the operator's.
  `TestCheck_RefusesLandingHalts` gains a blocked Discussion-Bouncer status with a prefixed error, wanting the refusal ending with `HandBackWayForward`, and a blocked Discussion-Bouncer status without the prefix, still eligible.
  What it bounds: only the prefixed Bouncer halt is withheld from the arbiter and the re-step; every other blocked stop stays eligible as today.
- In `circling.go`, the header, the group's Long help and `circlingWayNot` add the `decision` escalation (a review finding only the run's parent can settle) to the causes listed; the verbs record it like any other, and `continue` keeps its hard-cap refusal.

**ImpactSummary:** One refusal bit in arbiterstop and one condition in batten's hand-back seam, plus doc and help text; no other eligibility rule changes.
