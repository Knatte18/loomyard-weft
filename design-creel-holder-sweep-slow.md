# creel removed-holder sweep took 80 s instead of about 60

`creel-holder-sweep-slow` · Note · bug, creel

Imported from [issue #685](https://github.com/Knatte18/loomyard/issues/685).

Seen: strands frank and gus were removed at 08:48:03; their held messages went `returned`/`undelivered` at 08:49:23, 80 s later. The outcomes themselves were correct.

Expected: per sandbox scenario C11, within about 60 s.

Found by the creel stage 1 re-test of 2026-10-16 (sandbox scenario C11).