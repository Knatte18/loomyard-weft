# The bridge poll fails to skip its own posts and posts a name-clash notice for each

`bridge-own-post-not-skipped` · Note · bug, slack

## Seen (operator, 2026-11-26, first live run Neve to osl46)

Neve's orch posted `[lym-bridge] from=neve::ly:orch to=osl46::norce:orch id=2650` into `#bridge-neve-ly-orch--osl46-norce-orch`.
Right after, Neve's own relay (`neve::ly:SLACK`) posted in the same channel:
"a message names neve, a name this machine holds or held, as its sender; it was not delivered; way forward: ask the operator to give the other machine a name this machine never held".
So the bridge poll read its own post as inbound mail.

## Likely cause

`bridgeReader.handle` in `internal/slackengine/bridgein.go` skips its own post only when `message.User == r.botUserID`.
The relay posts with a custom `username` (`api.go`, the sender's `Username`, shown as `neve::ly:orch`).
Slack stores such a post as a `bot_message` carrying `bot_id` and `username` but no `user`, so the check never matches.
The post then reaches `handleEnvelope`, whose from-clash check fires.
The unit tests likely give the fake message a `User`, which is why they passed.

## Idea

Skip a message whose `bot_id` is this machine's own bot (from `auth.test`, beside the bot user id), and keep the `user` check as well.
A test feeds a message shaped as Slack returns a `username`-override post: `subtype: bot_message`, `bot_id`, no `user`.
Every outbound bridge post on both machines makes the false notice today, so it hides real clashes.
