# Keep the parent's satisfied PR-Review answer through later failed asks

Under autonomy, `lym loom pr-continue` and the PR-Review row end a review only on the parent's satisfied answer at the current head.
`landingshed.ReadPRReviewStanding` judged that answer from the run's latest PR-Review ask alone.
A later ask at the same head that ended `undelivered`, `no-parent` or `timeout`, or that still awaited an answer, hid a satisfied answer the parent had already given there.
`pr-continue` then refused with "no satisfied answer from your parent stands", and the review could not end.

Closes #213

## What changes

- `ReadPRReviewStanding` first looks for the latest answered, uncleared PR-Review ask at the head.
  When its report holds no findings, the standing is satisfied, whatever later asks at that head did.
  When it holds findings, a later answer has replaced the satisfied one, and the standing falls back to the latest ask as before: awaiting or ask-again.
- `TestReadPRReviewStanding` gains rows for a satisfied answer followed by failed asks, by an open ask, and by an answer with findings.
  It also gains rows for an open ask after findings and for a cleared satisfied answer.
- The autonomous PR-Review landing block tells the session that a satisfied answer at its head stands through a later failed or pending ask.
  The `landingshed` package doc states the same rule.

## Design decisions

- The fix lives in the one standing read, so `pr-continue`, the PR-Review row's judgement and `parent-review ask`'s awaiting check all follow it.
- Item 2 of the issue, a parent verb to settle an ask it never received, is already covered by 3e8a5eebf:
  `parent-review answer` answers a latest PR-Review ask that ended `no-parent` or `undelivered`.
  With this change a latest `timeout` ask needs no verb either, so no `--head` flag is added.
- Item 3, a tmux-tier test of `parent-review ask` from a live non-orch pane, leaves this run by the parent's recorded decision.
  It is filed as the board note `pr-ask-live-pane-test`.

## Merge danger

**Door:** two-way
**Blast radius:** landing
