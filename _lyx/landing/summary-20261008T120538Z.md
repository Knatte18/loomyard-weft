# Review loops converge from round 2 and carry their open findings

Review segments no longer circle to their budget on fresh edge-case keys.
On 2026-10-08 the Discussion reviews of three runs spent all five rounds and escalated on `budget`, although the last reviewer approved with no BLOCKING finding and every round only raised new keys.
The judge now rules `CONVERGED` from round 2 on once the latest round has no BLOCKING finding and no MEDIUM-or-worse `design` finding on a key already open in an earlier round.
The findings a segment closes on are written into the decision record, so the next producer or the PR-Gate reviewer sees them.
The change also drops the default round budget to three, adds per-segment review and fix models and a `fix_start` option, and lets `lyx loom resume` act on a recorded circling decision.

Closes #446

## What changes

- **Convergence rule** (`internal/shedadapters`): `decisionRuleMarker` renders a lighter rule for round 2 and later.
  Round 1 keeps today's rule, so every first-round fix still gets one fresh review.
  The round facts gain an earlier-open list: every key `open` in at least one earlier ledger.
  An earlier ledger that is missing or does not parse renders a parse-error line naming its round, and the rule text then tells the judge to rule by round 1's rule, so a degraded read never empties the guard.
  The CIRCLING bullet and checkpoint are unchanged, and the judge stencil's "fixed findings never count as convergence" sentence now states the new bound.
- **Carry-over into `## Open risks`** (`discussionparser`, `shedadapters`): when a Bouncer settles a judged round as `Done` (`CONVERGED` or a circling `accept`), Go writes one entry per segment between `<!-- lyx:carry-over <segment> -->` and `<!-- /lyx:carry-over <segment> -->` marker lines.
  It lists every open ledger entry at MEDIUM or worse plus unlabelled open entries, with key, class, severity, the rounds it was open in, and anchor-relative review and fixer-report paths.
  A later `Done` in the same segment replaces the entry, a ledger with nothing to carry removes it, and a skipped review leaves it alone.
  The write is committed through its own seam, which commits the decision record alone, and runs before Approve and Commit.
- **Consumers** (stencils): Plan-Write and rework treat an entry as findings to check against the code, never as scope.
  Describe turns each listed Plan-Review or Webster-Review finding into an item to check by hand.
  The Plan-Review rubric keeps an entry out of its answer key.
- **Budget**: `review_max_bounces` in the `loom.yaml` template goes from 5 to 3, and `review_circling_checkpoint` stays 3.
  The template also sets `review` to `opus[medium]` and `webster_review` to `sonnet[high]`.
- **Per-segment models**: `loom.yaml` gains `discussion_review`, `discussion_fix`, `plan_review`, `plan_fix`, `webster_review` and `webster_fix`.
  An empty value inherits `review` or `fix`; a set value replaces it for that segment and is validated through `modelspec.Parse` at load.
- **`fix_start`**: `parallel` (default, today's behavior) or `after-review`.
  With `after-review`, `burlerengine` starts the fixer only once the reviewer half has ended and the review is accepted; a reviewer that fails starts no fixer.
  The option is run-wide and reaches the engine through `RunOpts`.
- **`lyx loom resume`** (`loomcli`): over a run awaiting at a Bouncer row with a recorded, unsettled circling decision, it wakes the parked driver like a halted run.
  With no recorded decision it refuses and names `lyx loom circling accept` / `continue` and then `lyx loom resume`; PR-Gate keeps its approve/reject refusal.
  The circling verbs now print `"resume": "lyx loom resume"`, and the escalation brief names it.
- **Docs**: `PATTERN-review-round` is amended (a fix of a key first raised in the closing round may converge, carried to `## Open risks`) in `PATTERN.md` and `pattern/PATTERN-review-round.md`.
  The `doc.go` files, `docs/overview.md`, the refusal spec and the shed-recipe spec follow.

## Design decisions to review

- The carry-over text is Go-owned: a later edit between the markers is overwritten, and every byte outside them is kept.
  Markers count only inside `## Open risks` and outside fenced blocks, so marker text in prose elsewhere is ordinary text.
  The write refuses, and leaves the file unchanged, when the section is missing or its marker set is not zero or one well-formed pair per segment.
  It also checks the result before replacing the file: same H2 sequence, one well-formed pair for its segment, every other segment's entry byte-identical.
- Every judge-written field is rendered on one line with backticks escaped, so a finding key cannot open a heading, a fence or a marker.
- Accepted bound: the lighter rule lets a round converge over fixes of fresh MEDIUM `design` keys that no fresh reviewer has seen.
  It never lets through BLOCKING, a finding mapped to an earlier-open key, a finding in a round whose earlier ledgers did not all read, or any round-1 fix.
- A Plan-Review or Webster-Review entry is first checked at Describe, after the code is built, because Webster reads no decision record.
  PR-Gate's rejection into rework is the way back.
- The judge maps findings to keys, and no Go check reads that mapping: a judge that gives an already-open finding a fresh key converges a round the rule means to continue.
- With the budget and the checkpoint both 3, CIRCLING never changes a default run's outcome; lower the checkpoint for an earlier circling escalation.
- `fix_start` and the new model keys live in `loom.yaml`, not `burler.yaml`, which is a seed-only lens library.
- Out of scope: auto-accept at the round cap, lens fans, per-segment judge models or budgets, and `loom approve` / `reject`.

## To check by hand

- Run a real loom task with `fix_start: after-review` and confirm the fixer pane starts only after the review file exists.
- Escalate a Discussion round, record `lyx loom circling accept`, then run `lyx loom resume` against the live parked driver, also under a watching batten run; none of this ran against a real driver.
- Settle a real round and inspect the `## Open risks` entry and its commit in the decision record.
- The test-budget file `cmd/lyx/testdata/test-budget.yaml` was touched for new tests; check the entries look right.
