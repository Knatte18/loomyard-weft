## Goal

Dobby reminds the operator on Slack of a run waiting at PR-Review or Sampler-Session

Takes: slack-relay's presence reader, remind_after_min loader and SLACK program mailbox; dobby's probe-tick child readings and its tolerant state record; pauserecord's resumed entries
Delivers: a once-a-minute check in every worktree's dobby that keeps each PR-Review or Sampler-Session wait's start in dobby's state record and posts one reminder line per interval to <shortname>:SLACK while presence is slack
Exports: pauserecord's latest-resumed-entry lookup for a slug

## Scope

The part changes these packages:

- internal/dobbyengine (about 250 changed lines)
- internal/beamcli (about 40 changed lines)
- internal/pauserecord (about 40 changed lines)
- internal/slackengine (about 20 changed lines)

## Decisions

### D8 — Operator-wait reminders from dobby while presence is `slack`

- Each worktree's dobby, the prime's included, checks once a minute, whatever the presence, each running child whose current row is PR-Review or Sampler-Session, and reminds only while presence is `slack`.
- The wait's start is the later of the time dobby first reads the child running at that row with no pause covering it, whatever the presence, and the `resumed_at` of the latest resumed entry covering the child in the hub's pause record (`internal/pauserecord`), so a pause and resume dobby does not observe, between two checks or while it is down, restart the wait too.
  Dobby keeps it in its state record under the worktree's `.lym/beam/` directory, tolerant JSON, and clears it when the child leaves the row, stops running or a pause covers it.
- When the child has waited at that row for the reminder interval, the `slack` config key `remind_after_min` with a default of 30, dobby posts to `<shortname>:SLACK` one line naming the slug, the row and the minutes waited, and posts again each further interval while the wait lasts.
- A post creel refuses is logged at Warn and tried at the next check.
- Beam-Await, though in `loomshed.OperatorWaitRows`, is not reminded, since it waits on the beam's members as well as the operator.
- Reminders stop when presence goes back to `terminal`; on a return to `slack` the next check reminds of a wait already past the interval at once.

Rationale: dobby already reads each running child's status on its ticks, and reminding is mechanical work, which dobby owns (`dormant-orch`).
Bound: a pause or a resume restarts the wait's clock, and a restarted dobby keeps the wait's start from its state record; the last-reminder time is kept in memory, so a restarted dobby may remind once early.

## Constraints

- Slack facts (docs.slack.dev, fetched 2026-11-05):
  - `conversations.history` returns top-level messages; a thread reply is returned only from `conversations.replies`. A message's `thread_ts` equals its own `ts` on a thread parent and differs on a reply (docs.slack.dev/messaging/retrieving-messages).
  - `conversations.history`'s `oldest` is exclusive unless `inclusive` is true; messages come newest first, with `has_more` and a `response_metadata.next_cursor` to page; a `429` carries a `Retry-After` header in seconds (docs.slack.dev/reference/methods/conversations.history).
  - Slack sends a URL in a message's text as `<url>` or `<url|label>` and a user mention as `<@U…>` (docs.slack.dev/messaging/formatting-message-text).
  - The message event carries `user` for the author, and `bot_id` or subtype `bot_message` on messages an integration posted (docs.slack.dev/reference/events/message).
  - The 2025-05-29 rate-limit change (1 request a minute, 15 messages) applies to commercially distributed apps outside the Marketplace; "internal customer-built applications are not impacted" and keep Tier 3, 50+ requests a minute (docs.slack.dev/changelog/2025/05/29/rate-limit-changes-for-non-marketplace-apps). The limit is per app per workspace, so every hub's relay on the machine's one app (D1) shares it; D4's 5-second floor is 12 requests a minute per hub, and four hubs fit at 48 requests a minute.
- `go.mod` carries no Slack or WebSocket library today; the relay calls the Web API over `net/http`, adding no dependency.
- The relay's loops join the declared list in `cmd/lym/daemonwakeups_test.go`.
- The PATTERN IDs this record names are read under their cluster IDs if `pattern-distill` lands first; `PATTERN-slack-boundary` then joins the cluster that guards agent messaging, as a change to it, when one exists at Plan time.

## Auto-mode assumptions

- One channel per hub (D1), polling (D4), the `@<slug>` routing prefix (D4), and reminders for PR-Review and Sampler-Session every 30 minutes (D8) are auto-picks; no operator answered.

## Open risks

- `pr-review-wait-silent` asks for D8's PR-Review reminder; once this task lands, the parent should mark that note covered or remove it.

## Acceptance criteria

- D8: with presence `slack`, a child at PR-Review or Sampler-Session for the reminder interval, counted from the time dobby first read it running at the row with no pause covering it, gets one reminder per interval to `<shortname>:SLACK`; a child paused and resumed at PR-Review, whether or not dobby read the pause, gets no reminder before a full interval after the resume; a child at Beam-Await, and any child while presence is `terminal`, gets none.
