# Lens review

Every fork returned a report.

- criteria-coverage, MEDIUM, D6 distinct `unresolved` count untested — fixed: card 2 adds the begin-batch row "an unresolved ref two cards list is counted once", which wants `unresolved=1`.
- criteria-coverage, MEDIUM, "one quarry call" untested — fixed: card 2 moves `KickstartPack`'s body into `kickstartPack`, which takes a `resolve` func, and adds a `TestKickstartPack` subtest whose stub `resolve` records its calls; the subtest wants exactly one call over the distinct refs.
- criteria-coverage, MEDIUM, pack-on clauses (offset/limit, whole-file read of an edited file, the kept sentences, D4's unresolved and edited-file reads) not named — fixed: card 3 claims each clause as its own `wantAll` string.
- criteria-coverage, LOW, multi-line method uncovered — fixed: card 1 adds `T.LongMethod` to the fixture and the row.
- criteria-coverage, LOW, pack card order could diverge from pointer order — fixed: card 2's `buildKickstartPack` walks `batch.Cards` for the re-parsed cards (the filter loop at `internal/websterengine/beginbatch.go:404-409` walks `fresh.Cards`).
- row-reachability, MEDIUM, grouped `const` row passes untrimmed — fixed: card 1 requires a multi-line initializer for every `var` and `const` the row edits, grouped included.
- row-reachability, LOW, heading needle relies on the fixture's sibling temp dirs — fixed: card 2 names the source of the standalone display and asserts the pointer bullet beside the heading.
- forced-edits, LOW, `TestPlanDirDisplay`'s `//testtiming:keep` reason goes stale — fixed: card 2 lists `internal/websterengine#TestPlanDirDisplay` and rewords the reason.
- forced-edits, LOW, stale locate-opening needles in `TestRenderPrompts` — fixed: card 3 swaps them for the reworded opening.
- test-economy, LOW, the standalone `TestRenderPrompts` row duplicates the begin-batch `cardHeadings` check — fixed: the row is dropped.
- test-economy, LOW, the recovery byte-identity row duplicates the existing recovery row's render — fixed: folded into the existing row.
- test-economy, LOW, the switch-off and language-none needles in `TestBeginBatch_KickstartPack` repeat the render row — rejected: the language-none acceptance criterion is reachable only through begin-batch, since the render row only sees a nil pack.
- test-economy, LOW, `wantCount` repeats the render row's per-card rendering — fixed: `wantCount` is dropped, and the begin-batch rows assert the distinct log counts.
