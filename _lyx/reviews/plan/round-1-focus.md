---
round: 1
focus:
  - "Fidelity: every Decision D1-D8 and every Constraint in `_lyx/discussion/decision-record.md` is carried by some card, and no card adds scope the record does not license; check D6 first, where the overview's `peers-geometry` decision departs from the record's `internal/cliwire` wording."
  - "Attack surface, cards 6, 7, 9: the agent's verdict approves or rejects a review through Go; does each card's Intent state what an approve or reject verdict can now skip or let through, and what bounds it (fallback path, closed attempt, `skip_alignment`)?"
  - "Attack surface, card 1 and card 9: the new `skip_alignment` and timeout keys and the PR-side off switch; does the Intent state what they bypass and what bounds them?"
  - "Reachability of refusals: for each refusal or guard branch cards 1, 3, 5, 6, 8 and 9 prescribe (timeout-below-wait check, `noReviewer` in `final`, closed-attempt drop, `RemoveRejection` failure, missing pair or branch in peers), is there a named input and a named test that reaches it, and does no upstream check reject that input?"
  - "Card order: card 6 (gate) and card 7 (wiring) use `alignment.Session` (5), `RecordApproval`/`RecordRejection` (8) and `ResolveAlignment` (1); does any card depend on a later card's symbol, notably 6 or 7 on 8, or 4 on 2?"
  - "Verify coverage: the `## verify:` section runs `go test ./...` and `go test -tags integration ./...`; check that every package a card targets, including `internal/testkit/envkit`, `internal/loomcli`, `internal/battencli` and `internal/landingshed`, has tests the section runs, and that section order matches the plan template."
  - "Test economy and granularity: cards 4, 5, 9 plan many test functions; name any new test whose behavior an existing test already covers, and any card that splits a private supporting type or inseparable constructor into its own card."
  - "Edit-card glyphs and re-sign arrows: for each `Edit` card (1, 3, 6, 7, 8, 9, 10), do targets list member glyphs for the symbols its Intent names, and carry the re-sign arrow where the Intent changes a member's parameters, results or receiver (for example `Aligner` seams on `GateConfig`, `RecordApproval` and `RecordRejection`, `Deps` in landingshed)?"
  - "`Custom` and `ImpactSummary`: any `Custom` card whose targets a multi-label combination could express, and any `Edit` or `Delete` card whose `ImpactSummary` restates Intent rather than giving a blast-radius conclusion."
---

Seed pass; no prior round exists.
`lens-review.md` in the plan directory is not a card and not in the Card Index, so it is outside the card checks.
Each entry names where to look and the question to settle, and none is a verdict.
