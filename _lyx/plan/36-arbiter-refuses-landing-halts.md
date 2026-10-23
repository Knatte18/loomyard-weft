# Card 36 — arbiter-refuses-landing-halts

**Create:**
- `plan:internal/arbiterstop#TestCheck_RefusesLandingHalts` -> `func TestCheck_RefusesLandingHalts(t *testing.T)`

**Edit:**
- `internal/arbiterstop#Stop`
- `internal/arbiterstop#Eligibility`
- `internal/arbiterstop#Check`
- `internal/loomcli#arbiterFixture.haltAtVerifyRow`
- `internal/loomcli#TestArbiterVerb_SpawnPath`

**Uses:**
- `plan:internal/landingshed#HandsBack`

**Extends:**
- `internal/arbiterstop/doc.go`

**Intent:** `arbiterstop.Eligibility` refuses the landing halts, its refusal ending with `HandBackWayForward` (D11), so the arbiter verb, the Go driver's escalation fact and batten's eligible-stop hold all hand them to the orch and batten's notice is not held for an arbiter.
`Stop` gains `LandingHalt`, which `Check` sets from `landingshed.HandsBack(status.CurrentProducer, status.Error)` for a blocked status; `Eligibility` refuses a blocked stop with it set, naming the landing halt, before its other blocked-stop rule.
A tackler or arbiter resume would grant fresh fixer attempts or three more reroutes, or spend another Opus fixer on an escalation no decision answered, which is why these halts go to the orch.

`TestCheck_RefusesLandingHalts` pins a refusal ending with `HandBackWayForward` for each landing halt (a fixer-spent and an escalation halt at Merge-In, a budget-0 over-cap halt at Merge-In, a fixer-timeout halt at Finalize and a reroute-cap halt at Finalize), and eligibility for a Webster reroute-cap halt and an ordinary blocked halt.
The arbiter verb's spawn-path test gains a landing-halt row refused with `HandBackWayForward`, through the fixture's halt helper, which takes the halt reason.

`doc.go` names the refusal.

**ImpactSummary:** One eligibility refusal added; its readers are the arbiter verb, the driver's escalation fact and batten's hold reading, which all answer through `Check`.
