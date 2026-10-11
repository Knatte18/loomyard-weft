# Planning burler-split-budget: delete-before-reference matches method names as bare words

`delete-before-reference` matches a deleted method glyph's member name textually, so deleting `probingBurlerRunner.Run` or `shedfake.BurlerRunner.Run` was flagged against every later card whose edit code holds any `.Run(` call, `doc.go` prose included, and `BurlerProducer.Call` drew an informational match on an unrelated `.Call` in `internal/proc/namedjob_windows.go`.
About 220 blocking findings came from a handful of deletes; the only way through was to move every deletion into one last card and to delete a test helper type by listing its whole file instead of its methods, which loses the `caller-uncovered` check on those methods.

A match qualified by the receiver type, or restricted to selector expressions whose receiver resolves to the deleted type when a type load answers, as `caller-uncovered` already does, would have left the plan's natural order (retire the old producer right after the switch) valid.
