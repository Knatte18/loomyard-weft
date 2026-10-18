# creel: carry non-agent mail in the typed line, read prints plain text

A program's notice used to cost the orch three steps: a contentless `[creel] N unread` reminder, a `lyx creel read`, and a JSON envelope with fields it never uses.
Now mail from any sender but an agent strand is carried in full in the line creel types, and `lyx creel read` prints plain text.

## What changes

- **Carried mail.** A message from a program or `ly:EXTERNAL` whose body is at most 4 KiB is typed as `[creel] #<id> from <sender>: <body>` when the receiver is idle, and is journaled read afterwards.
  Line breaks and other control characters in the body become one space.
  The id leads the line, so a sender that takes replies (the orch watcher) is answered with `lyx creel post --reply <id>` without a read.
- **Reminder.** The count, the listed ids and the backoff cover only the unread mail that is not carried, so a receiver whose unread mail is all carried gets no reminder.
  Mail from an agent strand, and carried-eligible mail over the cap, keeps the contentless reminder.
- **Delivery order.** One delivery per probe: the first due inject, else the oldest carried message, else the reminder.
  Under a hold only the hold owner's mail is typed.
  A busy send leaves the message unread for the next probe; a send that did not land marks it read, as an inject that did not land is not retyped.
- **`lyx creel read`.** With no filter it returns every unread message, oldest first, in one call; `--from` narrows it to one sender, and `--all` and the request's `all` field are gone (a client that still sends `all` reads everything, since unknown keys are ignored).
  Output is a header line (`#<id> from <sender>`, with the message it answers or the receiver that returned it unread when those apply, and `posted <time>` only when the message is over five minutes old), the reply command, then the body; messages are separated by a blank line, and `nothing unread` prints when there is none.
  `--envelope` prints the previous JSON envelope unchanged, for programs.
- The Go client's read, the orch role stencils' mail steps, the sandbox creel suite (`tools/sandbox/SANDBOX-CREEL-SUITE.md`), `docs/overview.md` and the `creelengine`, `creelcli`, `orchcli` and `orchengine` docs follow.

## Design decisions

- **Agent mail is never carried.** An agent's text typed into another agent's pane would let it pose as an instruction; the reminder carries no sender text, and agent mail keeps it.
  A program name is accepted only from an orch pane or the Go client, and `ly:EXTERNAL` has no strand, so neither poses that risk.
- **No "takes replies" flag.** Case does not tell whether a program answers (the watcher does, batten does not), so every carried line shows its id and creel does not need to know.
- **Retype windows.** A daemon crash between the send and the read record types that message once more at the next start.
  A message a `read` holds reserved is neither typed nor journaled read by the delivery; if that read is released rather than acknowledged, the line is typed once after the release, and the journal still holds a single read.
- **Journal unchanged.** The existing `read` event records a carried message, so `lyx creel log` keeps the full history, including the metadata `read` no longer prints.
- **Flag name.** The JSON flag is `--envelope`, not `--json`, which the CLI's own-JSON-flag test pins.
- **Out of scope.** How batten or any other program words its notices, and notices posted under a strand claim, which stay agent mail.

## Check by hand

- A receiver that relied on the `[batten notice]` mail being readable after delivery now finds it only in `lyx creel log`; the orch role stencil names that place.
- The tmux and LLM smoke tests (`smoke_tmux_test.go`, `smoke_llm_test.go`) are updated to the new read output but are not part of the default test run; run them if you want the end-to-end check.
