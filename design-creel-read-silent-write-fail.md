# lyx creel read to an unwritable stdout exits 1 with nothing on stderr

`creel-read-silent-write-fail` · Note · bug, creel

Imported from [issue #681](https://github.com/Knatte18/loomyard/issues/681).

Seen: `lyx creel read > /dev/full 2>err` exits 1 and `err` is empty. The mail correctly stays unread (the #660 fix holds).

Expected: a short error on stderr, since the JSON error envelope cannot be written to stdout either.

Found by the creel stage 1 re-test of 2026-10-16 (sandbox scenario C4).