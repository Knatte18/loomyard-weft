# creel: a daemon that fails to start leaves verbs with only 'no daemon answers'

`creel-daemon-start-error` · Note · bug

Imported from [issue #661](https://github.com/Knatte18/loomyard/issues/661).

Found by the creel stage 1 audit (board note `creel-test`), in a sandbox hub, with `lyx` built from `main` at e131c7625.

## What was seen

When the creel daemon cannot start, every verb reports only that no daemon answers, after about 30 seconds, and the reason is nowhere to be found.
The daemon is spawned detached with null stdio (`internal/creelcli/spawn.go`) and writes no log file, so its own error is lost.

Seen with a malformed line in the middle of a journal day file, which the design says fails the load naming the file and line.
`lyx creel inbox` printed: `creel: no daemon answers on /tmp/lyx-1000/creel-ctest-41ca7726.sock after starting one and 6 connects: dial unix ...: connect: no such file or directory`.
Only running the hidden `lyx creel daemon --hub-path <hub>` in the foreground showed the real error: `creel journal: .../2026-10-14.jsonl line 3 is not a valid record (...); way forward: repair or remove that line by hand, then start the daemon again`.
`_board/.lyx/logs` held no creel entry.

## Steps

1. Stop the sandbox hub's daemon (replace its binary).
2. Add a non-JSON line in the middle of a day file under `_board/.lyx/creel/journal/`.
3. Run any verb, for example `lyx creel inbox`.

## Expected

A verb whose daemon fails to start reports the daemon's own error, or names a log file that holds it, so the operator can follow the "way forward" the daemon already writes.
