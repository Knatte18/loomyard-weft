---
round: 3
focus:
  - "D10 in decision-record.md: the landing join. Settle whether it names the commit that actually performed the parent-side merge, given that the child's Finalize does the merge (internal/loomcli/landingdeps.go) and batten's Land row only pulls the prime, deploys and prunes the board (internal/battenshed/doc.go). Check the join's stated bound, that no first-parent commit is claimed twice, how the checkpoint source joins to a parent-branch commit when the checkpoint is written before the merge, and that `hand` is derived from who performed the merge and not from the Land row state."
  - "D4, D5 and D9 in decision-record.md: the tolerance rule for parse-from-bytes functions. Settle whether one rule says each owning package gains a separate tolerant function for the report while strict existing ones stay unchanged (shedrun.ParseSeed in internal/shedrun/seed.go decodes with DisallowUnknownFields and is called by the live ReadSeed), and whether D4's rule for which fresh seed is a slug's own tag gives an absent batten source when no fresh seed falls between the previous tag and this one."
  - "Notes for the plan writer, D7 and the Acceptance criteria in decision-record.md: whether the verdict-read pointer agrees with the tolerant-reader rule (alignment.ReadVerdict in internal/alignment/records.go is path-based and uses KnownFields(true)), whether D7's halt-note presence test counts a run with Tier 2 on and no notes as zero stops since git does not track an empty friction dir, and whether each classification rule that carries a requirement has a criterion a test can fail on."
---

Focus targets the Decisions that carried the BLOCKING and gating findings in round 2.
