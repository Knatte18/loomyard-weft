# Card 17 — arbiter-drops-verify-rule

**Edit:**
- `internal/arbiterstop#Stop`
- `internal/arbiterstop#Eligibility`
- `internal/arbiterstop#Readers`
- `internal/arbiterstop#Check`
- `internal/arbiterstop#NewReaders`
- `internal/loomcli#arbiterFixture`
- `internal/loomcli#arbiterFixture.haltAtVerifyRow`
- `internal/loomcli#arbiterFixture.deps`
- `internal/loomcli#TestArbiterVerb_SpawnPath`

**Extends:**
- `internal/arbiterstop/doc.go`

**Intent:** Remove `internal/arbiterstop`'s rule for a Merge-In or Publish verify halt (D11): a halt no longer belongs to batten's retry, so the refusal it gave goes.
`Stop` drops `VerifyHaltNamesTarget`, `Eligibility` drops its refusal, `Readers` drops `VerifyRecord` and `Head`, and `Check` and `NewReaders` stop reading the failure record and HEAD.
A blocked stop is eligible as before until card 36 adds the landing-halt refusal.
`TestEscalationEligible` builds its readers without the two dropped fields and needs no change, since it never set them.
In loomcli's arbiter verb tests, `arbiterFixture` drops its verify record and `deps` stops filling the two readers; `haltAtVerifyRow` keeps halting the run at a row but leaves no record, and `TestArbiterVerb_SpawnPath`'s verify-halt rows, which expected a refusal, now expect the spawn every other halted blocked stop gets.

`doc.go` drops the verify-halt rule.

**ImpactSummary:** One eligibility rule and two readers go; the package's callers (the arbiter verb, the driver's escalation fact and batten's hold reading) call `Check` and `NewReaders` unchanged.
