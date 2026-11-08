# Slack: a reply is not forwarded when mail waits at turn end, and a forward can reach Slack late

`slack-stop-hook-drops-reply` · Task · bug, slack, claude, creel

Depends on: [`test-binary-fork-guard`](design-test-binary-fork-guard.md)

## Seen (orch, 2026-11-08)

The orch's final reply to an operator message from Slack never reached Slack; the reply to the turn after it did, about 30 seconds late.
Creel's journal, times UTC:

- 15:47:59.07 the operator's Slack message (#1092) is posted to `ly:orch`; 15:47:59.22 the relay posts `[slack presence] presence is now slack` (#1093), while the orch's turn on #1092 runs.
- 15:48:02 the orch's Stop hook hands #1093 to the session; no post from `ly:orch` to `ly:SLACK` is logged for the #1092 reply.
- 15:48:05 the next turn (on #1093) ends and its reply is posted (#1094); the relay reads it only at 15:48:34, when the operator next acted.

## Cause read from the settings

The orch's Stop hook (claudeengine settings) runs `lym creel hook-read claude && exit 0` when the strand's creel flag is set, before it reaches `lym slack hook claude`.
So whenever mail is waiting at turn end the hook hands the mail over and exits, and the turn's final reply is never forwarded.
An operator message from Slack always brings a presence notice right behind it when presence was `terminal`, so the first reply after switching to Slack is always lost.

## Idea

- Run the Slack forwarding on every Stop, before or regardless of the creel hand-over, so a waiting mail never drops a reply.
- Find why the relay read #1094 about 30 seconds after it was posted, when #1089 was read in 0.2 seconds; a forward should reach Slack at once.
- A presence notice caused by the operator's own Slack message need not wake the orch at all.
