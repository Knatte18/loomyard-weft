# Card 1 — interrupt-policy-words

**Move:**
- `internal/loomshed#InterruptPolicyReinvoke`
- `internal/loomshed#InterruptPolicyHandback`

**Edit:**
- `internal/loomshed#InterruptPolicies`
- `internal/loomshed#TestInterruptPolicyFor`
- `internal/loomshed#TestInterruptPolicies_ClosedVocabulary`
- `internal/loomshed#loomshedAllowedImports`
- `internal/arbiterstop#Eligibility`
- `internal/loomcli#decideDrive`
- `internal/loomcli#TestDecideDrive`
- `internal/loomcli#TestArbiterVerb_SpawnPath`
- `internal/loomcli#TestStatusCmd_EnvelopeKeySet`
- `internal/loomcli#TestStepEnvelope_NextInterruptPolicyMatchesTable`
- `internal/loomrecipe#TestInterruptPolicies_MatchEngineIdentity`

**Intent:**
Move the two interrupt-policy words out of loom into the generic verb layer, so the Go driver's rule table can name them once it leaves loom (D1 bars `internal/sheddriver` from importing `loomshed`), and batten can answer them from its own `InterruptPolicyFor` hook (D2).

Move `InterruptPolicyReinvoke` and `InterruptPolicyHandback`, names and values unchanged, into `internal/shedverbs/spec.go` beside `internal/shedverbs#Hooks`, whose `InterruptPolicyFor` field doc then names them as the two words a hook answers, the empty string staying the no-entry signal.
Their doc comments move with them and become recipe-neutral: reinvoke means the row's adapter reattaches to a live agent so a re-step never double-spawns, handback means the supervisor hands the stop back instead of re-stepping.
loom's per-row table `InterruptPolicies` stays in `internal/loomshed/interruptpolicy.go` and maps its rows to the `shedverbs` constants; its file header and table doc name them there.
`loomshed` gains its first `internal/shedverbs` import, so `loomshedAllowedImports` in `internal/loomshed/seam_enforcement_test.go` admits it with a one-line reason (the two policy words, no behaviour); `shedverbs` imports no `lyxcwd`, so the told-geometry membership holds.

Every reference retargets from `loomshed.` to `shedverbs.`: the arbiter's eligibility, the rule table in `internal/loomcli/driverules.go`, and the tests listed.
`loomshed`'s in-package tests take the qualified `shedverbs.` names.
No value, branch or envelope key changes, so the existing assertions are the coverage of this pure move.

**ImpactSummary:** two constants change package; eleven referencing members across loomshed, arbiterstop, loomcli and loomrecipe retarget with no behaviour change.
