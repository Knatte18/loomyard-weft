# creel log --id omits the reminded events that list that id

`creel-log-id-reminders` · Note · bug, creel

Imported from [issue #701](https://github.com/Knatte18/loomyard/issues/701).

Found in the creel stage 1 re-test of 2026-10-17 (main at 58a7feec7, dev build, sandbox hub lyt).

Seen: mail #2 to lyt:eve was reminded three times, and the journal holds `{"event":"reminded","to":"lyt:eve","unread":[2]}` records (and later `"unread":[2,44]`), which `lyx creel log --to lyt:eve` shows.
`lyx creel log --id 2` returns only the `posted` event; none of the reminders appear.
The same holds for every message: `log --id 4` showed `posted` and `read` but not the `reminded` event between them.

Cause: since #664 a `reminded` record carries its ids in `unread` and has no `id`, and the `--id` filter matches `id` only.

Reproduce: post a message to a live agent strand, let it be reminded (a `reminded` record appears in `log --to <strand>`), then run `lyx creel log --id <id>`.

Expected: `log --id <id>` includes each `reminded` event whose `unread` lists that id, so one message's history (posted, reminded, read) can be read in one call; or the --id help says reminders are not included.