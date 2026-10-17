# creel CLI output fixes: lost success envelope, doubled prefix, log --id reminders

`creel-cli-output-fixes` · Task · bug, creel · running · Webster-Burler-Fix

## Goal

Fix three output bugs in `internal/creelcli` found in the creel stage 1 re-test of 2026-10-17 (main at `58a7feec7`, dev build, sandbox hub `lyt`).
Operator decision 2026-10-17: fix these as one loom task, then go on to creel stage 2.

## 1. A success envelope that cannot be written is lost (#699)

Seen: with stdout unwritable, every creel verb whose answer is a success exits 0 and prints nothing on stderr; the answer is lost.

- `lyx creel post lyt:dave "x" > /dev/full` exits 0, stderr empty; the message (#907) is stored, but its id never reaches the caller.
- `lyx creel inject lyt:sink "x" > /dev/full` exits 0, stderr empty, and the line is typed.
- `lyx creel log > /dev/full` and `lyx creel inbox > /dev/full` (from a strand pane) exit 0, stderr empty.

A refusal to the same stdout is handled (#681): it writes `creel: <error>` to stderr and exits 1, and `read` leaves the mail unread.

Cause: the shared success path in `internal/creelcli/cli.go` (`output.Ok(out, res.fields)`) drops the write error; only `writeRefusal` and `read`'s `writeOk` check it.

Expected: when the success envelope cannot be written, a short `creel: ...` line on stderr and a non-zero exit, as for a refusal; for post and inject the stderr line names the id, since the message or inject was already stored.

## 2. The stderr fallback prints the prefix twice (#700)

Seen: a refusal whose JSON envelope cannot be written to stdout (the #681 fallback) prints its message with the prefix doubled:

- `lyx creel inbox > /dev/full` from a terminal: `creel: creel: an EXTERNAL caller holds no mail; ...`
- `lyx creel post lyt:nosuch x > /dev/full`: `creel: creel: unknown receiver "lyt:nosuch"; ...`

`read > /dev/full` in a strand pane prints `creel: creel read: could not write the messages, so they stay unread: ...`, which reads as intended.

Cause: `writeRefusal` in `internal/creelcli/cli.go` prepends `creel: ` to a message that the engine's errors already open with `creel: `.

Expected: one prefix, as the creelcli `doc.go` line "writes `creel: <error>` to stderr" describes.

## 3. `log --id` omits the reminded events (#701)

Seen: mail #2 to `lyt:eve` was reminded three times, and the journal holds `{"event":"reminded","to":"lyt:eve","unread":[2]}` records, which `lyx creel log --to lyt:eve` shows.
`lyx creel log --id 2` returns only the `posted` event; `log --id 4` showed `posted` and `read` but not the `reminded` event between them.

Cause: since #664 a `reminded` record carries its ids in `unread` and has no `id`, and the `--id` filter matches `id` only.

Expected: `log --id <id>` includes each `reminded` event whose `unread` lists that id, so one message's history (posted, reminded, read) reads in one call.

## Reproduce

In a hub with live strands: `lyx creel log > /dev/full; echo $?`, the same for post, inject and inbox; `lyx creel inbox > /dev/full` from a terminal; post to a live agent strand, let it be reminded, then `lyx creel log --id <id>`.
