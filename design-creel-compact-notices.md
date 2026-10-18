# [WS 001] Creel: program notices in the reminder line, plain-text read

`creel-compact-notices` · Task · enhancement, creel, workspaces · running · PR-Gate

## Seen (2026-10-18)

- A one-line batten notice now takes three steps for the orch: a contentless `[creel] N unread` reminder, a `lyx creel read`, and a JSON reply carrying `messages`, `body`, `from`, `envelope`, `id`, `kind` and `posted`.
- `lyx creel read` reads only the oldest sender's messages by default, so two senders take two reads.
- The receiver needs the body always, sender and id only when a reply is wanted (`post --reply <id>`), the time only when a message is old (withheld test posts arrived a day late), and envelope and kind never; the creel journal (`lyx creel log`) keeps the full metadata for debugging.

## Operator decisions (2026-10-18)

- A program sender's notice (an upper-case sender such as `ly:BATTEN`, which takes no reply) is carried in the reminder line itself, such as `[creel] ly:BATTEN: test-budget-remove: landed; newly ready: batten-start-speed`, needs no read, and is marked read on delivery.
- Mail from an agent keeps the contentless reminder (operator decision 2026-10-17); `lyx creel read` prints plain text for every sender at once: id and sender with how to reply, then the body, with the time only when the message is more than a few minutes old.
- JSON only behind a flag, for programs.
