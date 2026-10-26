# Support log — weft-suffix-card-slug-audit

## Interview

A darn run holds no interview; the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- Skip the value of `--card` in `ReferenceRule.MatchPath`: rejected because a card id also appears in other commands a fork runs, such as paths under webster's state directory and grep patterns, and each would still trip the audit; it would also teach the audit one CLI's flag grammar.
- A new check ID such as `card-slug-weft-suffix`: rejected because it is the same rule and matcher as `card-fabric-reference`, and a new ID adds a spec row and a stencil mention for no gain.
- Import `weftname` into planparser and test `strings.HasSuffix`: rejected because planparser's seam deliberately keeps fabric naming out.

## Review rounds

_No rounds yet._

## Question ledger

- Refuse at the plan gate or relax the audit? Self-pick: refuse at the plan gate, since it covers every spelling of the card id.
- Should a done card in a rework be exempt? Self-pick: no; the check takes no done set today and the case is rare, recorded as an open risk.
