---
dispositions:
  - id: F2
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F9
    disposition: fixed
---

## Changes

- F2 (`c7d2671d`): `DroppedCreates` in `internal/discussionparser/append.go` lost its unreachable malformed-heading error and the doc sentence promising it; it reads the title from the parsed heading.
  The way forward in `internal/runplan/postbatch.go` ends "re-run the verb that failed", so it holds for the weaver call sites too.
- F5 (`58eb8ab2`): `namedByAmendment` returns false for a plan file that holds no card.
  `planparser` exposes the check as a predicate over its own overview name, so no second literal exists.
  `TestCheckReviewRound` gained a row where an amendment naming the overview is refused with `does not list`.
- F9 (`4c899857`): `TestBurlerReviewEntry_ProfileMapping` gained the `AmendmentReaderReachesOnlyThePlanReviewRow` subtest.
  It lays out a complete round 1, lists an added entry in an amendment record, and asserts `Round.AmendingTexts` holds that entry for `Plan-Burler-Review` and none for `review-row`.
- Self-check follow-up (`7742b7c5`): see below.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: `doc.go` and the `RoundContext.AmendingTexts` field comment said "a file" where the exemption covers plan card files — fixed as proposed; the `doc.go` sentence also drops the stale "plain substring" claim, since the match is a whole token.
- fix-proposal, LOW: the overview-only predicate left `amendments.md`, `lens-review.md` and lens reports exempt by the same route — fixed with a wider change than the review proposed:
  `planparser.IsNonCardFileName` covers every non-card plan file and replaces the overview-only predicate;
  the index-file check reuses it, so the two lists cannot drift.
  Two rows, for `amendments.md` and `lens-review.md`, pin it.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
