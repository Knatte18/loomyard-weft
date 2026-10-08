# Review loops converge from round 2, carrying fresh findings

Review segments no longer circle to their budget on fresh edge-case keys.
From round 2 on, a segment converges once its latest round carries no BLOCKING finding and no MEDIUM-or-worse `design` finding on a key already open in an earlier round.
Round 1 keeps today's rule, so every first-round fix still gets one fresh review.
The findings a round closes on are carried into the decision record, so the next producer and the PR-Gate reviewer see them.

Closes #446

## What changes

- **Convergence rule.**
  `decisionRuleMarker` in `internal/shedadapters/bouncerprompt.go` renders the round-1 rule or the lighter rule by round, as it already does for the CIRCLING bullet.
  The round facts gain an earlier-open list: every key `open` in at least one earlier ledger.
  An earlier ledger that is missing or fails to parse renders a parse-error line naming its round, and the lighter rule then falls back to round 1's rule.
  The judge stencil's "Fixed findings never count as convergence" is reworded to match.
- **Carry-over.**
  When a Bouncer settles a judged round as `Done`, Go writes that segment's entry into the decision record's `## Open risks`.
  It lists each open ledger entry at MEDIUM or worse, and each open entry with no class or severity, with key, class, severity, rounds open and anchor-relative review and fixer-report paths.
  The write is a new `discussionparser` function beside `AppendDecision`, committed alone through a Bouncer seam wired for all three review segments.
  The Plan-Write and rework stencils treat an entry as findings to check against the code, never as scope.
  The Describe stencil turns each Plan-Review or Webster-Review finding into an item to check by hand.
  The Plan-Review rubric keeps an entry out of its answer key.
- **Budget.**
  `review_max_bounces` in the template drops from 5 to 3; a spent budget escalates to the parent as before.
- **Per-segment models.**
  `loom.yaml` gains `discussion_review`, `discussion_fix`, `plan_review`, `plan_fix`, `webster_review` and `webster_fix`.
  An empty value inherits `review` or `fix`.
  The template sets `review: opus[medium]` and `webster_review: sonnet[high]`.
- **Fixer start.**
  A new run-wide `loom.yaml` key `fix_start` takes `parallel` (default, today's behavior) or `after-review`.
  With `after-review`, the fixer starts only once Go has accepted the review and written the ready marker.
  A reviewer that ends without an accepted review starts no fixer.
  Any other value is refused at load.
- **`lyx loom resume` and circling.**
  Over a run awaiting at a Bouncer row with a recorded, unsettled circling decision, `lyx loom resume` wakes the parked driver.
  With no recorded decision it refuses and names the circling verbs and `lyx loom resume`; at PR-Gate it refuses as before.
  The circling verbs' `resume` field and the escalation brief now name `lyx loom resume`.

## Design decisions to review

- The lighter rule departs from `PATTERN-review-round`'s "fixed findings never converge on their own".
  `PATTERN.md` and `pattern/PATTERN-review-round.md` are amended in this change: a fix of a key first raised in the closing round may converge, carried to `## Open risks`.
- A round-2-or-later round can converge over a fix of a fresh MEDIUM `design` key that no fresh reviewer has seen.
  It cannot converge over a BLOCKING finding, a key in the earlier-open list, a round whose earlier ledgers did not all read, or any round-1 fix.
- The judge maps findings to keys, and no Go check reads that mapping.
  A judge that gives an already-open finding a fresh key converges a round the rule means to continue.
- Each segment owns at most one entry, delimited by `<!-- lyx:carry-over <segment> -->` and a closing marker.
  A later settle replaces the bytes between that segment's markers and keeps every byte outside them.
  Markers count only inside `## Open risks` and outside fences; marker text elsewhere is prose.
  A malformed marker set, a missing `## Open risks` heading, or a result that fails the re-read check (same H2 sequence, one well-formed pair per segment, other entries byte-identical) leaves the record unchanged and fails the settle.
  A skipped review leaves the segment's entry as it is.
- A Plan-Review or Webster-Review entry is first checked at Describe, after Webster has built the code, because Webster reads no decision record.
  PR-Gate's rejection into rework is the way back.
- With `review_circling_checkpoint` and `review_max_bounces` both 3, CIRCLING never changes a default run's outcome; lowering the checkpoint gives an earlier circling escalation.
- Auto-accept at the round cap was dropped; a spent budget still escalates.

## To check by hand

- Run a real review segment past round 2 and confirm the `## Open risks` entry is written, committed and then replaced on a later `Done`, and that `lyx loom validate-discussion` still passes on the record.
- Run with `fix_start: after-review` and confirm the fixer pane starts only after the review is accepted, and that its first `lyx burler await-review` returns at once.
- Escalate a run to circling, record `lyx loom circling accept`, and confirm `lyx loom resume` wakes the parked driver.
- Deploy the stencil changes with a production deploy: the board copies of `landing-template-describe`, `loom-template-plan`, `loom-template-rework`, `loom-rubric-plan-review`, `bouncer-template-judge` and `bouncer-template-escalation` log a drift warning until then.
