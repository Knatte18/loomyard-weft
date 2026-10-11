# Sweep tests that rely on a closed listener socket file

`reed-socket-test-fd-sweep` · Note · enhancement, reed

Seen in shuttle-startup-ready (PR #557, 2026-10-11): `TestRemoveStaleSocket` in `internal/reedengine` failed at Publish verify because a parallel test's forked child inherited the closed listener's fd, so the "stale" socket still answered.
Commit `ea4e53819` fixed `TestRemoveStaleSocket` and `TestRemoveSocketFileOnceGone` by building the stale socket from a datagram socket that never listens (`bindStaleUnixSocket`).

Idea: sweep other tests that build a stale socket by closing a listener and move them to `bindStaleUnixSocket`.