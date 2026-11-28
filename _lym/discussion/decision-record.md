## Goal

```markdown
Imported from [issue #294](https://github.com/Knatte18/loomyard/issues/294).

## What happened

An origin that is missing on disk is not treated as unreachable:

- A removed bare repository used as origin is classed `other` by the push failure classification, not as transport.
- A remote pointing at a missing directory makes the origin reconcile return an error instead of reporting origin unreachable.

Tests of an owed push or an unreachable origin had to use a refused connection URL instead.

## Why it is abnormal

An origin that cannot be reached for a path reason, such as a missing or unmounted directory, takes a different route from one that refuses a connection.
It may therefore stop a run where an owed push would be expected.

## What to do

Decide how a missing local remote should be classed.
Either treat it as unreachable in both places, or document that only network failures count.

Provenance: task `push-retry-not-halt`, notes `06-pusher-target-kinds.md`, `12-finalize-local-landing.md`.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - change-description-omits-affected-callers (class consistency, severity MEDIUM, open in rounds 1)
  - unconfigured-remote-name-reads-unreachable (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
