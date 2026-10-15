# creel: read does not show that a message was returned by a removed holder

`creel-read-returned-mark` · Note · bug, creel

Imported from [issue #663](https://github.com/Knatte18/loomyard/issues/663).

Found by the creel stage 1 audit (board note `creel-test`), in a sandbox hub, with `lyx` built from `main` at e131c7625.

## What was seen

A message that the removed-holder sweep returned does not say so when it is read.
The design says each unread message of a gone holder goes one step back "and is marked with the receiver that ended without reading it" (`internal/creelengine/doc.go`, "Removed holders").
The fold sets `Message.EndedUnreadBy` and `EndedUnreadAt`, and the journal's `returned` record holds the receiver, but `messageFields` in `internal/creelcli/cli.go` never prints them.
The return also strips the last forwarder from `path`, so the read shows a bare message.

## Steps

1. `ctest:dave` posts "sweep test" to `ctest:carol`; carol forwards it to `ctest:stuck2`.
2. `lyx reed remove <stuck2 guid>`; about a minute later the journal shows `{"event":"returned","id":908,"to":"ctest:carol","receiver":"ctest:stuck2"}`.
3. carol runs `lyx creel read --from ctest:dave` and gets `{"body":"sweep test","from":"ctest:dave","id":908,"kind":"mail","posted":...}`, with no word that it came back from stuck2.
4. The same holds for a sender who gets their own message back (dave's #32 returned from a removed strand reads like a note to self).

## Expected

`read` prints the returned mark (for example `ended_unread_by` and its time), so the reader knows the message bounced and from whom.
