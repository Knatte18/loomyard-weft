# creel journal: a bad last line ending in a newline is skipped, not cut, and the next daemon start refuses the journal

`creel-journal-torn-newline-line` · Note · bug, creel

Imported from [issue #677](https://github.com/Knatte18/loomyard/issues/677).

Seen: with `{"event":"posted","at":"2026-10-16T08:45:1` plus a newline as the last journal line, the creel daemon logged `skipping a torn last line left by a crash ... line=101` and appended new records after it. At the next start that line sat in the middle of the file and the daemon refused it: `line 101 is not a valid record (unexpected end of JSON input); way forward: repair or remove that line by hand`. Creel was down until the line was removed by hand.

Expected: the skipped line is cut, as a torn tail without a newline already is (`cutting a torn last line ... bytes=42`), so a later start does not fail on it.

Found by the creel stage 1 re-test of 2026-10-16 (sandbox scenario C10).