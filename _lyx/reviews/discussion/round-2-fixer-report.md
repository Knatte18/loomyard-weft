# Round 2 fixer report — discussion

I checked each finding against `contracts/specs/loom-plan-spec.md` (the card-type table and the `Move` accepted false positive), `contracts/stencils/burler/burler-step-3-fix.md`, `contracts/stencils/bouncer/bouncer-template-escalation.md` and `internal/websterengine/cardgate.go`, and every premise held.
`lyx loom validate-discussion` passes after the fixes.

## Fixed

- **F1 (MEDIUM):** D6 gains a `Bound:`: a `Move` of a `go.mod` is not counted, since its destination is prose.
  For a card at or after the `Move`, D7 and D8 can differ between the plan gate and dispatch, the same blind spot the spec accepts for `path-missing`, and the author closes it with a `Rename` pair.
  D6's rationale now claims sameness only for the changing card and later cards, and says why earlier cards are harmless.
  D8's bound and rationale name the exception.
  In the support log, Q9 names the bound, and counting a `Move` source as a `Delete` is a rejected alternative.
- **F2 (MEDIUM):** D9's rationale no longer says the fixer disputes a blocked fix; a blocked fix leaves the premise true, so it cannot be disputed.
  D9's Decision now says the `**Fix:**` line is a proposal: the fixer may apply any other admissible fix, and with none it defers the finding, naming the constraining check.
  D9's bound states that the finding then blocks convergence and reaches the parent session and the operator through the judge's escalation.
  Q10 in the support log matches.
- **F3 (LOW):** D1's bound now says the card gate catches root-module assignment sites in untagged code, and tagged ones only in the card's own directories.
  The plan-level `## verify:` gate catches tagged sites elsewhere later, and no gate builds a site in an other-GOOS file.

## Disputed

None.

## Deferred

None.
