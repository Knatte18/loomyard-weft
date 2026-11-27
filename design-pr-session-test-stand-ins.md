# PR-Review session tests still hand-write reed, creel and session stand-ins

`pr-session-test-stand-ins` · Task · enhancement, testkit, landing · starting · Seed-Child

## Seen

Code-Review round 1 of the task that restores the PR-Review agent session (#282) raised a BLOCKING PATTERN finding: the session tests and `envkit` stand in for repository modules.
The fixer removed `fakePRSessions` and envkit's `noopSessions` and runs the real `shuttleengine.Runner` over `shuttlefake`, adding `shuttlefake.ResumeEngine`.
Still hand-written: the `SessionLive` closures over the kit's strand table and in `envkit`, the `RegisterSession` closures, and the `MailAgent` closures, one of them returning a made-up "no such strand" error.
No kit holds a recorded session stand-in, and a real reed needs tmux while these tests are tier0.
The parent accepted the round with this open, to land the urgent fix.

## Idea

Give testkit a recorded reed and creel session stand-in usable at tier0, and move the PR-Review session tests onto it.
