# PR-Review session tests run over a real creel daemon and the recorded tmux kit, which learns to replay pastes

`pr-session-mail-stand-ins` · Task · enhancement, testkit, landing, reed, creel · running · Friction-Reflect

## Seen

The run that gave testkit a recorded tmux stand-in under reedengine left two stand-ins out of its scope by the parent's decision:
the `MailAgent` closures in the PR-Review session tests, one returning a made-up "no such strand" error,
and the session stand-ins in `envkit`.
`MailAgent` posts through creelcli's `GoClient`, which talks to a creel daemon it spawns, so no real creel path runs at tier0.

The landed recorded tmux kit (`internal/testkit/tmuxrec`) neither records nor matches a call's stdin.
Each paste names a fresh random buffer in its arguments, so a replayed paste matches no recorded one and fails as unanswered.
No current recording holds a `load-buffer` call, so a scenario that sends text to a pane cannot use the kit.

## Change

- Move the `MailAgent` rows to a tier1 loomcli test over a real creel daemon.
- Teach `tmuxrec` to record and match a call's stdin, and to match a paste's buffer name by pattern rather than literally, with a recording that pastes and a replay test over it.
- Move `envkit`'s session stand-ins onto the recorded tmux kit, using paste replay where they send text.
