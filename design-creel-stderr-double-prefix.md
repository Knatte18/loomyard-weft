# creel stderr fallback for an unwritable refusal prints the prefix twice: creel: creel:

`creel-stderr-double-prefix` · Note · bug, creel

Imported from [issue #700](https://github.com/Knatte18/loomyard/issues/700).

Found in the creel stage 1 re-test of 2026-10-17 (main at 58a7feec7, dev build, sandbox hub lyt).

Seen: a refusal whose JSON envelope cannot be written to stdout (the #681 fallback) prints its message with the prefix doubled:
- `lyx creel inbox > /dev/full` from a terminal: `creel: creel: an EXTERNAL caller holds no mail; ...`
- `lyx creel post lyt:nosuch x > /dev/full`: `creel: creel: unknown receiver "lyt:nosuch"; ...`
`read > /dev/full` in a strand pane prints `creel: creel read: could not write the messages, so they stay unread: ...`, which reads as intended.

Cause: `writeRefusal` in internal/creelcli/cli.go prepends `creel: ` to a message that the engine's errors already open with `creel: `.

Reproduce: from a terminal in a hub, run `lyx creel inbox > /dev/full`.

Expected: one prefix, `creel: an EXTERNAL caller holds no mail; ...`, as the creelcli doc.go line "writes `creel: <error>` to stderr" describes.