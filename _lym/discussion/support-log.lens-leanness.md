```yaml
lens: leanness
outcome: reported
```

Task: review the decision record and support log through the leanness lens, looking for content to move or cut.

**Findings**

1. MEDIUM — D2 Rationale. It names a rejected alternative inside `## Decisions`: "a content marker in the record would let a writer forge the reset".
   - Evidence: the compaction rule says Decisions carries Decision and Rationale only, and that rejected alternatives go to the support log. The support log's Rejected alternatives already holds this exact item.
   - Fix: keep the positive half ("only the verb writes the trailer") and cut the marker clause.

2. MEDIUM — Scope "Out". Most bullets carry their rejection reason, which is deliberation the support log already holds.
   - Evidence: the support log's Rejected alternatives repeats each reason: the body is not in the record format, replace covers edit and remove, discussion without a Discussion.
   - Completeness test: the exclusions themselves bind the plan writer, especially "fixers keep never touching them", so keep each bullet and drop only its "because" clause.

3. LOW — Scope "In". It restates D1 to D6 one by one, so the design is said twice.
   - Fix: shorten it to one line pointing at the Decisions, or cut it to the items Decisions does not already state. Today there are none.

4. LOW — D2. Its last paragraph lists every caller of the history seam: the discussion gates, Darn-Write's guard, validate-discussion and `add`'s precondition.
   - Evidence: the rubric's do-not-flag list leaves call-site enumeration to Plan-Write, and the same point is made again in Notes for the plan writer, bullet 2.
   - Fix: keep the requirement ("every caller of the seam reads the bounded history") and cut the list, or keep only the Notes version.

5. LOW — Constraints. The bullets for `PATTERN-sole-parsers`, `PATTERN-agent-addressing` and `PATTERN-cli-cobra` restate D1 and D4 almost word for word.
   - Fix: shorten them to the entry name plus "complied with by D1 and D4", or cut them. `PATTERN-run-records` already points to PATTERN changes.

6. LOW — Auto-mode assumptions. The first bullet (one verb over edit and remove) repeats Scope Out and the Q2 ledger entry. The fourth (warning, not refusal) repeats D5's "It never refuses".
   - Fix: keep only the assumptions that the Decisions do not already state as decided, such as base-hash ergonomics and folding before approval.

7. LOW — Open risks, third bullet (an operator hand-editing `board.json` skews the times). This is speculative and applies to every board field, so it adds no risk specific to this task. Cut it.

8. LOW — D1 Rationale. It restates the bullets above it: "base hash makes it safe" repeats the base-hash bullet.
   - Fix: keep only the new clause, "one whole-record verb covers edit, remove and fold".

9. LOW — D5 Bound. It overlaps the Rationale's issue #116 citation in emphasis only; this is fine. The issue citation belongs to the support log's interview rather than the record, so it could move. This is optional.

Nothing found that must move out wholesale. Every Decision carries requirements the plan writer needs.

Tool calls: none.
