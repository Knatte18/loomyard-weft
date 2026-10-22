# Round 2 fixer report — discussion

## Changes

All changes are in `_lyx/discussion/decision-record.md` unless noted.

- F1 (fixed as proposed): D7 makes the amendment record tracked content at `_lyx/loom/amendment.yaml`, committed through the fabric in the decision entry's `commitDiscussion` commit, removed inside Plan-Bouncer's commit seam and in PR-Rework's round commit.
  The Constraints bullet names the tracked path, and Open risks bullet 2 is removed.
  `support-log.md` gains Q15 recording the change.
- F2 (fixed differently): the settle record gains `round_started_at`, read from the review half's `start` in `round-<N>-usage.yaml`, with an acceptance criterion for a rework-exempt (round-0) generation.
  The proposal had the round-0 settle write its own settle time and had a missing start read as the settle time.
  I did not make that change: a round-0 settle via `approveWithoutReview`, which also settles a zero-allowance amendment generation (`internal/shedadapters/amendment.go:21,51`), would then remove the record with no round judged.
  A missing start read as the settle time would also strand an amendment written during a round whose start was not recorded: the guard would refuse it and nothing would remove it.
  Instead, a settle with no known round start admits the goto whenever the record exists and never removes the record.
  Plan-Review's skip seam answers false while the record exists, so the re-opened round of an exempt generation is judged.
- F3 (fixed as proposed): an append failure restores the amendment record to its bytes before the call, removing it only when the call created it.
  The criterion now says a failed second call keeps the first call's title.
- F4 (fixed differently): on a commit error the verb reads the committed amendment record and compares it with the bytes this call wrote.
  On a match it keeps both files and reports success with the error noted.
  On a mismatch it restores both files and gives the re-run way forward.
  When HEAD cannot be read it keeps both files and gives `lyx fabric commit` as the way forward.
  The proposal used `committedHoldsEntry`, which matches by heading (by, date, title), so a same-title same-day second call whose commit failed would read as landed.
  The call's nanosecond write time makes the byte comparison unique to the call.
  The landed-commit case is added to the acceptance criterion.
- F5 (fixed as proposed): the answer-key instruction now raises a finding per card the entry names, or, when the entry names no card, until some card carries it.
- F6 (fixed as proposed): D9 step 3 matches a `not_found` answer through its `Unit`; otherwise the location goes into `untraced`.
  The `untraced` definition and an acceptance criterion (unused-import build error) are updated.
- F7 (fixed as proposed): D9 step 3 reads targets through `RefMember`, and a target `RefMember` does not map compares as written, a `plan:` handle as its body.
  An acceptance criterion now covers a test created under a `plan:` handle.
- F8 (fixed as proposed): D9 drops the "no per-card commit" claim.
  It defines the batch range for done and for failed or in-flight batches, and adds a `later_range` field: from the recorded head of the highest-numbered matched batch that has one to the record's `Head`.
  The note renders it, and an acceptance criterion covers both ranges.
- F9 (fixed as proposed): D5's guess matches on a member glyph's last dotted segment.
- F10 (fixed as proposed): the amendment write time and `round_started_at` are recorded at nanosecond precision and compared as instants.
- F11 (fixed as proposed): D1 says two cards fencing a line for one entry ID make an edge whatever their labels.
  D4's rule text carries the exception, and the batch-order criterion covers it.
- F12 (fixed as proposed): D4's Rationale narrows the claim to the pair verdict and says each caller keeps its own overlap detection.
- F13 (fixed as proposed): the orch-stencil criterion is reworded to name the route.

## Self-check

- fix-consistency, LOW, the `PATTERN-tolerant-records` constraint omits `round_started_at`: fixed as proposed.
- fix-consistency, LOW, the `deriveEdges` plan-writer note omits fenced entry IDs: fixed as proposed.
- fix-way-forward, MEDIUM, an unknown round start strands a mid-round amendment: fixed with my own change, where no known start admits the goto and never removes the record.
- fix-way-forward, MEDIUM, a goto on an exempt generation can re-settle through the skip seam unjudged: fixed as proposed, with the skip seam answering false while the record exists (`loomshed.PlanReviewSkippable`).
- fix-claims, MEDIUM, `RefMember` has no answer for file or self-glyph targets: fixed as proposed.
- fix-claims, LOW, a zero-allowance amendment generation's round-0 settle was not covered: fixed, since every round-0 `approveWithoutReview` settle now records no round start.
- fix-claims, LOW, `_lyx/loom/` might be ignored in the weft: disputed, premise false.
  The weft checkout has no `.gitignore`, and `internal/loomengine/config.go:109` names `_lyx/loom` (`LoomDurableDir`) as the durable directory.
- fix-proposal, MEDIUM, a round-0 settle removes the record unreviewed: fixed with the fix-way-forward change above, where a round-0 settle records no start and never removes.
- fix-proposal, MEDIUM, the removal cannot be staged by the current pathspecs: fixed as proposed.
  `commitDiscussion`, Plan-Bouncer's plan commit and PR-Rework's commit pathspecs gain the file when it exists or is tracked at HEAD.
- fix-proposal, MEDIUM, `RefMember` drops non-member targets: duplicate of the fix-claims finding, fixed there.
- fix-proposal, LOW, a heading match misreads a same-title same-day call as landed: fixed with my own change, the byte comparison of the committed amendment record (see F4).
- fix-proposal, LOW, the later range is undefined when the highest matched batch has no head: fixed as proposed.

## Disputed

- Self-check fix-claims LOW (`_lyx/loom/` ignored in the weft): premise false; see the Self-check entry for the evidence.

## Deferred

None.
