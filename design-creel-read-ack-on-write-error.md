# creel: read marks mail read even when writing it out failed

`creel-read-ack-on-write-error` · Note · bug

Imported from [issue #660](https://github.com/Knatte18/loomyard/issues/660).

Found by the creel stage 1 audit (board note `creel-test`), in a sandbox hub, with `lyx` built from `main` at e131c7625.

## What was seen

`lyx creel read` marks messages read even when it failed to write them out, so the mail is lost.
The design says a read is acknowledged only once the client wrote the messages out (`internal/creelengine/doc.go`, "Mail"), and `Client.Call` acks only when its handler returns nil.
But the read verb's handler in `internal/creelcli/cli.go` (`printMessages` in `readCmd`) calls `output.Ok(out, fields)` and always returns nil, ignoring a write error.

## Steps

1. In a sandbox hub, post a message to a shell strand: `lyx creel post ctest:carol "w1 unacked test"`.
2. In carol's pane: `lyx creel read > /dev/full; echo rc=$?` prints `rc=0`.
3. `lyx creel inbox` in carol's pane shows no unread mail, and `lyx creel log --id <id>` shows a `read` event.

## Expected

A read whose output cannot be written exits non-zero and sends no acknowledgement, so the messages stay unread, as the design promises for a client that dies before writing them.
