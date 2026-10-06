# Fixer report — discussion round 1

All edits are in `_lyx/discussion/decision-record.md`; `support-log.md` is unchanged.

## Fixed

- **F1** — Decision 4: `unknown` by absence now requires a found listing; a transcript with no listing never classifies by absence, and a truncated listing can only cost one retry.
  Decision 6 and the unverified acceptance criterion now include "no skill listing found".
- **F2** — Decision 7 states that a timeout has no turn end and no transcript read, so every requested skill is skipped with cause `timeout`, with a bound for a skill that loaded before the hang.
  The Goal now reads "when that turn ends, or when it times out".
- **F3** — Decision 11: the retry step is its own persisted value distinct from old per-skill indices, and the step and retry list are written as one update.
  Decision 12: an old index colliding with the retry step's value is read as the pointer step.
- **F4** — Decision 13: the premise is worded as an assumption, with a bound stating what a wrong premise lets through.
- **F5** — Decisions 5 and 6: the retry names only skills the first turn classified `missing`, is classified with Decision 4's classes and causes, and falls under Decision 6's unverified degrade.
  The acceptance criterion now says "on the retry turn as on the first".
- **F6** — Decision 8: added a bound line for an unknown skill on an unreadable-transcript turn.
- **F7** — Decision 14: the rationale no longer re-argues the rejected short-timeout alternative, which stays in the support log.

## Deferred

Nothing deferred.
