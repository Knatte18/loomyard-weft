# Card 6 — driver-advisor-wording

**Prosa:**
- `contracts/stencils/shed/shed-template-driver.md`
- `contracts/stencils/loom/loom-template-discussion-advisor.md`

**Intent:** Two wording fixes from the stencil audit.

- D2 (F15): in `shed-template-driver.md`'s `## Stopping on a non-running state`, the clause after "A run halted at the gate re-runs only the gate on resume," becomes: a fix to a halted run is made by the run's own agents, through `reject` or a backward `goto`, which the driver reports and never runs, as `## Ways forward` says.
  The sentence names no hand commit and no operator push, and `goto` stays a verb the driver reports and never runs (`PATTERN-no-denied-recovery`).
- D3 (F16): in `loom-template-discussion-advisor.md`'s role paragraph, "read the record there for its format" is dropped; the sentence names the record's path so the advisor reads the draft once the chair has written it, to check it against PATTERN and the board, as its Step 3 says.
  The advisor's marker set is unchanged, so `TestStencils_TopLevelMarkers`' row for it holds.

The changes are static prose, which no test asserts.
