```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals behind F1 and F2, both fixed as proposed.

**F1 (the `PreflightStencils` doc line in `recipe.go`): no findings.**
- The added line says only the Darn-Write table is checked. It matches the wiring, which sets the closure in `wireDarn` only (`darnwiring.go:261`). `wiring_test.go:183` pins nil for a recipe with no Darn-Write row.
- The stated reason, that the other tables take call-time values Preflight does not have, is the reviewer's own account and I found nothing in context to contradict it.

**F2 (the `CheckMarkers` refusal text, its comment, the test and `doc.go`):**

1. LOW. `internal/preflightshed/doc.go:19-20` is now stale.
   - It says the stencils check refuses a marker "as after a version skew where a newer binary refreshed the stencils mid-run" and that "the reason carries the check's own way forward, a re-step with a binary that fills them".
   - The refusal now names a second cause (a blank value) and a second way forward (give the blank value text).
   - A reader of the preflight doc would think skew is the only cause.
   - Fix: in that sentence, add the blank-value cause and its way forward.

2. LOW. `internal/loomcli/darnwiring.go:224` has the same staleness.
   - It reads "so a stencil newer than the running binary halts the run at Preflight".
   - The check now also halts on a blank non-Optional value.
   - Fix: add "or a blank value" there.

3. The new wording is correct on the cases I built.
   - A blank value gives "because the stencils … are newer than this lym binary or a value is blank", which is accurate.
   - A skew gives the same text, and it is still accurate.
   - The test rows for a blank table value, a blank seat value and the block marker match on the unchanged `has marker(s) … that no value fills` prefix, so the rewording does not break them.
   - The one full-string row was updated to the new text.
   - `wiring_darn_test.go:195` matches only the seat, the stencil and the marker, so it is unaffected.

No tool calls made; my context held everything this lens needed.
