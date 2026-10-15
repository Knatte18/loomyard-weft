# Card 3 — Recovery prefix orients from the overview and the rules

**Edit:**
- `contracts/stencils/webster/webster-prefix-recovery.md`
- `contracts/stencils#wordingClaims`
- `internal/websterengine_test#TestRenderPrompts`

**Intent:** Carry decision D4: the cold recovery strand orients to what an in-session fork inherits, and reads its code per card.

In `webster-prefix-recovery.md`:

- The banner comment's "no codebase orientation, no plan framing, no constraints" becomes "no plan framing, no constraints, and no code, which no fork inherits either".
- Under `## Orient yourself before you touch anything`, step 1 (read `_lyx/plan/00-overview.md` in full) is unchanged.
  Step 2 "Orient to the codebase: read what your card(s) need, plus whatever the plan's decisions and any PATTERN entries above point you at — not a gratuitous full-repo tour." becomes: "Read the `pattern/` background files the constraints section above points at, when that section is present, choosing those that touch what your card(s) will change."
  A sentence after the steps reads: "The code your card(s) need you read per card, as the card steps below say, not as an up-front orientation step."
- "Do this BEFORE the card instructions below — they assume you already hold this orientation, exactly like an in-session fork already holds it from Master." becomes: "Do this BEFORE the card instructions below: it gives you what an in-session fork inherits from Master, the plan's framing and the repo's rules, and no code."
- Every other section, and every marker, is unchanged.

In `contracts/stencils#wordingClaims`, the `webster-prefix-recovery.md` row gains a `wantAll` of "the `pattern/` background files the constraints section above points at, when that section is present", "you read per card" and "what an in-session fork inherits from Master, the plan's framing and the repo's rules, and no code", and a `wantNone` of "Orient to the codebase" and "exactly like an in-session fork already holds it from Master".

In `internal/websterengine_test#TestRenderPrompts`:

- The row "recovery points a cold strand at the overview with PATTERN inactive" gains `contains` "what an in-session fork inherits from Master, the plan's framing and the repo's rules, and no code" and "you read per card", and `notContains` "Orient to the codebase".
  The fresh-read rule's "read — in THIS turn —" wording in the recovery prompt is already pinned by the row "recovery locates by quarry and carries no pack section", whose `contains` holds card 2's `prePackFreshRead`.
- The row "recovery carries the PATTERN overview when PATTERN is active" gains `contains` "the constraints section above points at, when that section is present".

**ImpactSummary:** One stencil, its claim row and two render rows; no marker, Go code or other prompt changes.

**Commit:** 3: webster: recovery orients from the plan overview and the rules
