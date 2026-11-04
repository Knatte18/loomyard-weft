## Goal

```markdown
## Seen (orch, 2026-11-01)

`landing-watch` (commit `c9d52bdb`) has dobby send the orch a line when a task with `notify_on_land` set lands.
Its commit lists known limits:

- A `notify_on_land` set after the run passed Land gets no line.
- A run that passes Land while dobby is down long enough for a fresh state record to be written gets no line.
- The line needs a live orch; after three failed posts it is dropped.

The notice was never run end to end against a live orch.

## Wanted (operator, 2026-11-04: close all three)

1. Setting `notify_on_land` on an entry whose run already landed answers at once: the board write says it landed, or the line is sent then.
2. A landing missed while dobby was down is still noticed: on start dobby finds `notify_on_land` entries that landed since it last ran (board done state, not only its own state record) and sends their lines.
3. A line the orch cannot take now waits in creel for the orch and reaches it when it comes back, never dropped after a fixed number of posts.

Show the notice end to end against a live orch once, in a test or by hand, and say which in the change description.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - done-entry-notice-ignores-owner-machine (class design, severity MEDIUM, open in rounds 1)
  - failed-clear-mode-read-restart-rows-missing (class scope, severity MEDIUM, open in rounds 1)
  - no-end-to-end-demonstration-of-landing-notice (class scope, severity MEDIUM, open in rounds 1)
  - sweep-returns-unread-mail-for-unlisted-hub-orch (class design, severity BLOCKING, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
