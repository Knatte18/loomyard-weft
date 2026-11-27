# PR-Review session tests still hand-write MailAgent closures and envkit session stand-ins

`pr-session-mail-stand-ins` · Note · enhancement, testkit, landing

Depends on: [`pr-session-test-stand-ins`](design-pr-session-test-stand-ins.md), [`push-retry-not-halt`](design-push-retry-not-halt.md)

## Seen

The run that gives testkit a recorded tmux stand-in under reedengine left two stand-ins out of its scope by the parent's decision:
the `MailAgent` closures in the PR-Review session tests, one returning a made-up "no such strand" error,
and the session stand-ins in `envkit`, whose `envkit.go` the running push-retry-not-halt also changes.
`MailAgent` posts through creelcli's `GoClient`, which talks to a creel daemon it spawns, so no real creel path runs at tier0.

## Idea

Move the `MailAgent` rows to a tier1 loomcli test over a real creel daemon,
and move `envkit`'s session stand-ins onto the recorded tmux kit.
