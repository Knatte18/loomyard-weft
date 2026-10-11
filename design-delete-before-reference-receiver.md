# Plan gate delete-before-reference matches deleted methods by bare name

`delete-before-reference-receiver` · Note · bug, quarry, loom

## Seen (2026-10-11, `burler-split-budget` Plan-Write)

The plan gate's `delete-before-reference` check matches a deleted method glyph's member name as a bare word.
Deleting `probingBurlerRunner.Run` or `shedfake.BurlerRunner.Run` flagged every later card whose edit code held any `.Run(` call, `doc.go` prose included, and `BurlerProducer.Call` drew a match on an unrelated `.Call` in `internal/proc/namedjob_windows.go`.
About 220 blocking findings came from a handful of deletes.
The writer moved every deletion into one last card and deleted a test helper type by listing its whole file, which loses the `caller-uncovered` check on those methods.
Plan-Write ran about 30 minutes and 192k output tokens, much of it on repeated `lyx loom validate-plan` passes.

## Idea

Match a deleted method by receiver type: count only selector expressions whose receiver resolves to the deleted type when a type load answers, as `caller-uncovered` already does.
The plan's natural order (retire the old producer right after the switch) then stays valid.
