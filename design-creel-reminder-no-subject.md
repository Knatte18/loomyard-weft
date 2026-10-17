# Creel reminders carry no message content

`creel-reminder-no-subject` · Note · enhancement, creel

## Seen (orch, 2026-10-17)

A demo post from `ly:orch` to a fresh Claude strand `ly:demo` was reminded as:
`[creel] 1 unread: #3 from ly:orch «Hello from ly:orch via creel. Please reply to this message with: lyx creel post » — run lyx creel read`.
This is the documented design (`internal/creelengine/doc.go`, the reminder paragraph): the subject is the body's first line, cut at 80 runes, inside « ».
A message without a separate first line therefore shows its content in the reminder, and the receiving agent can act on it without running `lyx creel read`.

## Operator view, not a decision (2026-10-17)

The reminder must not carry the message's content.
The receiver has to ask for it actively with `lyx creel read`.

## Idea

- The reminder names only the count, ids and senders, such as `[creel] 1 unread: #3 from ly:orch — run lyx creel read`.
- `lyx creel log` keeps its subjects, since only the operator and the orch read the journal.

## Open

- Whether the journal's `subject` field stays as the body's first line or goes too.
