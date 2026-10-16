# creel verb waits about 23 s to report a daemon that already exited at start

`creel-dead-daemon-slow-report` · Note · bug, creel

Imported from [issue #683](https://github.com/Knatte18/loomyard/issues/683).

Seen: against a malformed journal the creel daemon died at once (daemon log 10:46:01.395), but `lyx creel inbox` returned its error, which now quotes the daemon log as the #661 fix intends, only after about 23 s.

Expected: the verb reports the error as soon as the daemon it started has exited.

Found by the creel stage 1 re-test of 2026-10-16 (sandbox scenario C10).