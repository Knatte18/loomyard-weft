# Fixer report — discussion round 2

## Fixed

- F1 (BLOCKING): confirmed against `planparser.splitFraming` and `contracts/specs/loom-plan-spec.md`: `## verify:`, `## Shared Decisions` and `## Rename mechanic` follow the Card Index, and the frontmatter sits outside the framing.
  D7's second hash is now over the whole `00-overview.md` minus the Card Index section's lines (heading to next `## ` heading, as `planparser` bounds it), so frontmatter, framing and every plan-level section stay pinned.
  D7's decision, refusal wording and Bound now say "outside the Card Index nothing may change"; the acceptance line names the frontmatter, framing and `## verify:` as refused, and Scope "Out" now agrees with the mechanism.
- F2 (MEDIUM): confirmed: `batchCardIDs` compares `NN-slug` ids only, and the summary reaches Master's batch table (`render.go`).
  D7's Bound now pins a begun card's number and slug only, states that its one-line intent is free, and what that lets through (a reworded intent in a later Master render; the card file stays hash-pinned).
  The acceptance line says "a begun card's number or slug".
- F3 (LOW): D4's per-mode list gained the archive-only row for either mode (nothing discarded, nothing refused, tracked and untracked changes stay), and the envelope lists the left uncommitted paths on that path too; the D3 acceptance line names that listing.
- F4 (LOW): D6 states that a rebaseline naming a card with an existing entry resets its `rendered` to false, so the live recovery is forced failed and the next one renders the newer text; an acceptance clause pins it.
- Support log: round 2 entry under Review rounds, and Q15 reworded, Q16 and Q17 added to the question ledger.

## Disputed

None.

## Deferred

None.
