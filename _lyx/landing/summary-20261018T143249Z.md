# Creel: carry non-agent mail in its own line, plain-text read

A one-line batten notice used to cost the orch three steps: a contentless `[creel] N unread` reminder, a `lyx creel read`, and a JSON reply with fields it never needed.
With this change, mail from any sender but an agent strand is carried in full in the line creel types, and `lyx creel read` prints plain text for every sender at once.

## Carried mail

A message is carried when its sender is a program or EXTERNAL, any sender but an agent strand, and its body is at most 4096 bytes.
The delivery worker types it as its own line, `[creel] #<id> from <sender>: <body>`, after any due inject and before the reminder, one per probe, and only the hold owner's under a hold.
Every run of line breaks and other control characters in the body becomes one space.
The line opens with the id, so a program that takes replies, such as the orch watcher's note request, is answered with `lyx creel post --reply <id>` and no read; creel does not need to know which programs answer.
Once the send returns, typed or not confirmed, the store journals the message `read` by the receiver, so `lyx creel log` keeps its history; a busy send leaves it unread for the next probe.
The reminder's count, listed ids and backoff set cover only the rest: agent mail and bodies over the cap.

The code is `isNotice` and `composeNotice` in `internal/creelengine/reminder.go`, and the snapshot split, `worker.typeNotice` and `Store.markNoticeRead` in `deliver.go`.

## Read

`Store.Read` returns every unread message by default, and `--from` narrows it to one sender; the `--all` flag and the request's `all` field are gone.
`lyx creel read` prints each message as a header line and its body, with a blank line between two messages, and `nothing unread` when there is none.
The header is `#<id> from <sender>`, then `in reply to #N`, `returned unread by <receiver>`, and `posted <RFC 3339>` once the message is more than five minutes old by the CLI's clock.
It ends with `— reply with: lyx creel post --reply <id> <text>`.
Envelope, kind and path stay out of it, and `lyx creel log` keeps them.
`read --envelope` prints the old JSON envelope unchanged, and refusals keep the JSON error envelope every creel verb prints.

## Docs

`internal/creelengine/doc.go` and `internal/creelcli/doc.go` describe carried mail and the new read output, and `internal/orchcli/doc.go` and `internal/orchengine/doc.go` follow.
The orch role stencil's wake-up step tells a carried line from a reminder, and its After-landing fallback looks for a landed notice in `lyx creel log` first.
The two handoff stencils tell the orch to reply to the id the carried `[creel]` line opens with, not one `lyx creel read` prints.
The sandbox creel suite's scenarios post agent mail from a shell strand where they judge a reminder, and S9 also watches a terminal post being carried.

## Design decisions

- The parent's decision "Which mail needs creel read", recorded after Discussion, widens the board's program-only notice to EXTERNAL and puts the id in every carried line and every read header; this description follows it.
- A 4096-byte cap still applies to carried mail, so a long body does not flood a pane line; mail over it falls back to the reminder.
- The JSON form is `--envelope`, not `--json`: a verb's own `--json` shadows the root's global help flag, and the read-only classifier takes a trailing `--json` as a help form, which is wrong for a verb that marks mail read.
  `TestOwnJSONFlags` pins that set.
- Carried mail is journaled read after its send, not before: a busy send must not lose it, and the cost is one retype if the daemon dies between the send and the read record.
- A message a concurrent read holds reserved is left to that read, both when the snapshot picks carried mail and when `markNoticeRead` writes, so it is never journaled read twice.
- Carried mail is read, so a removed receiver no longer returns it.
  Rows that pin the returned-undelivered path for EXTERNAL or program mail now keep the receiver busy, so nothing is carried first, `TestDaemon_Protocol/PostWaitReplyIsAnsweredWhenTheMessageReturnsUndelivered` among them.
  `MailReturnsOneStepAtATimeThenIsUndeliveredAndAnswersItsWaiter` posts a body over the cap for the same reason.
  The CLI's reminded-log row posts from a strand pane.
- That sweep row's first event check also ignores `reminded` events: the worker types the returned mail's reminder with no clock advance, so that event could land before the check.
- The text rendering is pinned by an internal unit table, `TestWriteMessagesText`, because the CLI cannot post as a program and carried mail never reaches a read.
  The integration row covers plain text, the posting time past five minutes, `--envelope` and `nothing unread` end to end.
- The tmux and llm smoke tests post as EXTERNAL, so they now wait for the carried `[creel] #1 from ` line instead of a reminder; those tiers were not run here.

## Tests

`TestScheduler_Reminders` gains rows for each of these:

- a program's line carried with its id, read, and left out of the reminder beside strand mail;
- multi-line program and EXTERNAL mail carried flattened, with over-cap and strand mail reminded;
- a busy carried send left unread and a not-landed one read.

The hold row expects the owner's carried line and its over-cap mail reminded.
The read-set rows exercise the all-senders default and `--from`.
