## Goal

```markdown
Imported from [issue #153](https://github.com/Knatte18/loomyard/issues/153).

## What was seen

Only two reviews can be routed to the run's parent by setting: `pr_review_ask_parent` and `discussion_review_ask_parent` in `loom.yaml` or a task's `run_settings`.
The code review perch (Code-Burler-Review and Code-Burler-Fix) and the drift judge (Drift-Verify, Drift-Recheck) have no such key.
Their rulings reach the parent only when a finding is escalated as a decision call, or when an agent was told by hand to ask.
A parent that wants to see every ruling of a darn run, which has no Discussion, cannot get it by configuration.

## Wish

Add parent-review keys for the code review perch and the drift judge, parallel to `discussion_review_ask_parent`:

- `code_review_ask_parent`: each code review round mails the parent a `[code-review request]` naming the review and fixer report, and waits for `lym loom parent-review answer` (satisfied or a findings file) before the Bouncer judges the round, with a timeout key as `discussion_review_parent_timeout_min` has.
- `drift_judge_ask_parent`: the drift judge mails the parent its proposed rulings, every cell or a summary by case with the flagged cells named, and waits for the parent's agreement or changed rulings before it writes the judgement, with a timeout.

Both settable in `loom.yaml` (or `drift.yaml`) and in a task's `run_settings`.

## Workaround

Tell the run's agents by creel to ask the parent before they finish a round or write a judgement.



Out of scope: routing agent questions to the parent and parent notification on named rows are the separate notes `agent-questions-to-parent` and `parent-notify-on-row`.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-19): Opt-in parent-ask keys stay outside Guard

Decision: code_review_ask_parent, code_review_parent_timeout_min, judge_ask_parent and judge_parent_timeout_min stay settable in run_settings, as discussion_review_ask_parent and pr_review_ask_parent are. Amend PATTERN-config-layers with a written exception: a key that only adds an ask to the parent, and the timeout bounding that added wait, is not a Guard key, since unset or expired it leaves the run's own gates exactly as they are without it.

Rationale: The Guard rule exists to stop a local layer switching a check off. These keys can only add a check; their timeout only ends the added wait and never bypasses a gate the run has without them. The existing discussion and PR-review keys already work this way; the exception writes the precedent down.

### Added after Discussion (parent, 2026-11-19): Drift key names stand

Decision: drift.yaml's judge_ask_parent and judge_parent_timeout_min stand for the entry's drift_judge_ask_parent; no rename.

Rationale: The keys live in drift.yaml, so a drift_ prefix only repeats the file's name.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-2-review.md; fixer report: _lym/reviews/code/round-2-fixer-report.md.
  - code-ask-misses-rounds-own-review (class design, severity BLOCKING, open in rounds 1, 2)
  - judge-parent-block-embedded-in-go (class design, severity BLOCKING, open in rounds 2)
  - judge-parent-wait-outlives-judge (class design, severity MEDIUM, open in rounds 1, 2)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
