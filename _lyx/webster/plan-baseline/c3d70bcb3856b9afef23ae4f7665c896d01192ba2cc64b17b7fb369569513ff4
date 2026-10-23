# Card 4 — reroute-cap-prefix

**Create:**
- `plan:internal/shedengine#ReasonRerouteCapReached` -> `const ReasonRerouteCapReached = "reroute cap reached"`

**Edit:**
- `internal/shedengine#Shed.rerouteStuck`

**Extends:**
- `internal/shedengine/doc.go`

**Intent:** Name the reroute-cap halt's reason prefix as a constant, beside `ReasonBounceBudgetExhausted` and `ReasonPreconditionUnmet`, so the landing hand-back predicate (D11) matches it by identifier rather than by a copied literal.
`Shed.rerouteStuck` builds its halt reason from the constant; the text is unchanged, so the existing reroute tests keep covering it.
`doc.go`'s reroute paragraph names the constant.

**ImpactSummary:** A pure refactor of one reason literal into a constant; the halt text is byte-identical.
