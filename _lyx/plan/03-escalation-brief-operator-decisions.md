# Card 3 — escalation-brief-operator-decisions

**Edit:**
- `contracts/stencils/bouncer/bouncer-template-escalation.md`
- `contracts/stencils/bouncer/bouncer-fragment-decide-open.md`
- `contracts/stencils/bouncer/bouncer-fragment-decide-capped.md`

**Intent:** Make the Bouncer's escalation brief settle only on a recorded operator decision (D1) and decide nothing at cause `budget` (D2), one sentence per line, with no new marker, so `renderEscalation` in `internal/shedadapters` is unchanged.

`bouncer-template-escalation.md`:
- `## Materials` gains, first, the arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
  It gains, last, the support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.
- `## What to do` opens with the settle condition: settle only when a recorded operator decision answers the envelope's question directly.
  It then defines a recorded operator decision as one of: a passage of the task's board entry that the entry marks as an operator decision with a date, as `Operator decision (<date>)` does; a board note whose body states that it records an operator decision; a decision-record entry added with `--by operator` that is already in the committed decision record when the escalation arrives; or a Discussion answer that the support log's `## Interview` shows the operator gave.
  It names what never counts: an `Idea` or agent-suggestion passage, an operator view not marked as a decision, a decision-record entry added with `--by arbiter` or `--by parent`, a writer's auto-pick, and an `## Auto-mode assumptions` entry.
  It states that `--by` is an unverified label, since only `--by arbiter` is restricted, so a `--by operator` entry counts only when it was in the committed decision record before the escalation.
  It never settles a question whose answer widens the task's scope, and never accepts over an open design-wrong finding unless a recorded operator decision covers that finding.
- At cause `budget`, decide nothing: report back to your parent session which input could not be read.
- Step 1 becomes: record the operator decision you applied, citing where it is recorded, with `lyx loom decision add {{.slug}} --by parent --title … --decision … --rationale …`, run in `{{.worktree}}`, when the decision record exists; never record with `--by operator`.
  Without a decision record, name the applied decision and its source in your report back to your parent session.
- The closing paragraph stays: otherwise record nothing and report back to your parent session, which asks the operator; the run stays `awaiting`.
- The leading comment is unchanged in substance: Go still interpolates only paths, round, cause, segment and slug, plus the decide fragment.

`bouncer-fragment-decide-open.md` and `bouncer-fragment-decide-capped.md`: the lead-in "Record your decision on the segment" becomes "On the ground of the operator decision you applied, record the segment's outcome", keeping the rest of each line, so the capped fragment still reads "can only be accepted:" and offers no continue verb, which `TestBouncer_Escalation_AwaitsWithTheRecordAndLeavesItUntouched` asserts.

No test pins this static prose (`PATTERN-test-economy`); the existing brief render tests keep passing because the markers and the asserted phrases are unchanged.

**ImpactSummary:** Prose-only change to three Bouncer stencils with the same marker sets; `renderEscalation` and its tests are untouched.
