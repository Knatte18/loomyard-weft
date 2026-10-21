# reed watchdog never consumes the stop marker when its inotify watch fails to open

`reed-watchdog-inotify-fallback` · Note · bug, reed

Imported from [issue #783](https://github.com/Knatte18/loomyard/issues/783).

Seen (orch, 2026-10-21, OSL-1046, seven runs in flight): TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping fails in all three stop subtests on every rerun of `lyx gate test ./internal/reedcli` ("marker consumed at call 0"), blocking the verify of create-target-transient, whose diff does not reach internal/reedcli.

Cause, per the darn writer: fs.inotify.max_user_instances is 128 and the user holds about that many instances (VS Code, konsole windows per run, lyx processes, other desktop apps). The watchdog consumes the stop marker only through its filesystem watch (internal/reedcli/watchdog.go, the `events` branch); when the watch fails to open, `events` is nil and the marker is never read.

Expected: the watchdog falls back to reading the stop marker on its timer cycle when the watch fails to open, and says so in its log, so a full inotify table neither breaks stops in production nor fails the test. The test should not depend on a free inotify instance, or should report the exhaustion as its failure cause.

Workaround: raise fs.inotify.max_user_instances (sudo sysctl) and rerun the verify.