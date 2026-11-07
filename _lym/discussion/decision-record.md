## Goal

```markdown
## Seen (operator and orch, 2026-11-07)

`e25dcf34` renders `HEDDLE.md` beside `heddle.json` in the Bolt checkout, but only when `lym heddle set` or `lym heddle remove` writes the store, or heddle's Bolt replayer replays one.
This hub had no heddle write after the landing, so it has no `HEDDLE.md`; `lym heddle render` writes only the worktree's file targets and does not render it.
`HEDDLE.md` also lists the shipped entries, which change with a lym update while `heddle.json` stays as it is, so an existing file goes stale with no heddle write to renew it.

## Wanted

- A hub whose `HEDDLE.md` is missing, or no longer matches what heddle holds, gets it rendered and committed with no hand step: after the landing of this change, and after a lym update that changes the shipped entries.
  Find the hook that already runs at such a time, such as the landing deploy, the config reconcile or the Bolt sync, and render it there under the Bolt write lock; a render that would write identical bytes commits nothing.
- `lym heddle set` and `lym heddle remove` keep rendering it as today.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - failed-overview-render-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
