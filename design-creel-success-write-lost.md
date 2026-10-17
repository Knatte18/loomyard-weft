# creel verbs exit 0 with nothing on stderr when their success envelope cannot be written

`creel-success-write-lost` · Note · bug

Imported from [issue #699](https://github.com/Knatte18/loomyard/issues/699).

Found in the creel stage 1 re-test of 2026-10-17 (main at 58a7feec7, dev build, sandbox hub lyt).

Seen: with stdout unwritable, every creel verb whose answer is a success exits 0 and prints nothing on stderr; the answer is lost.
- `lyx creel post lyt:dave "x" > /dev/full` exits 0, stderr empty; the message (#907) is stored, but its id never reaches the caller.
- `lyx creel inject lyt:sink "x" > /dev/full` exits 0, stderr empty, and the line is typed.
- `lyx creel log > /dev/full` exits 0, stderr empty.
- `lyx creel inbox > /dev/full` from a strand pane exits 0, stderr empty.
A refusal to the same stdout is handled (#681): it writes `creel: <error>` to stderr and exits 1, and `read` leaves the mail unread.

Cause: the shared success path in internal/creelcli/cli.go (line 175, `output.Ok(out, res.fields)`) drops the write error; only `writeRefusal` and `read`'s `writeOk` check it.

Reproduce: in a hub with a live strand lyt:dave, run `lyx creel log > /dev/full; echo $?`, then the same for post, inject and inbox.

Expected: when the success envelope cannot be written, a short `creel: ...` line on stderr and a non-zero exit, as for a refusal; for post and inject the stderr line names the id, since the message or inject was already stored.