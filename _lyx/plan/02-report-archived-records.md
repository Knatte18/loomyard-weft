# Card 2 — report-archived-records

**Create:**
- `plan:internal/reportengine#parentReviewRound` -> `type parentReviewRound struct`
- `plan:internal/reportengine#parseParentReviewRound` -> `func parseParentReviewRound(number int, files map[string][]byte) (parentReviewRound, error)`
- `plan:internal/reportengine#parentReviewRoundNumber` -> `func parentReviewRoundNumber(dirName string) (int, bool)`
- `plan:internal/reportengine#parentReviewDirRel` -> `func parentReviewDirRel() string`
- `plan:internal/reportengine#alignmentAttempt` -> `type alignmentAttempt struct`
- `plan:internal/reportengine#alignmentVerdict` -> `type alignmentVerdict struct`
- `plan:internal/reportengine#parseAlignmentAttempt` -> `func parseAlignmentAttempt(files map[string][]byte) (alignmentAttempt, *alignmentVerdict, error)`

**Edit:**
- `internal/reportengine#RunRow`
- `internal/reportengine#RunRow.readParentReviewWakes`
- `internal/reportengine#RunRow.readAlignmentWakes`
- `internal/reportengine#settlerOf`
- `internal/reportengine#runAttempts`
- `internal/reportengine#RunRow.readSegments`
- `internal/reportengine#allowedImports`
- `internal/reportengine#parentRoundFiles`
- `internal/reportengine#parentRoundTimed`

**Intent:** D4: reports keep reading archived parent-review rounds and alignment attempts after both packages go, through a read-only parse kept inside `internal/reportengine`.
Add one file, `internal/reportengine/archivedrecords.go`, whose header says nothing writes these records any more and these readers only decode what archived runs hold.
It declares, as unexported copies of the on-disk shapes `internal/parentreview` and `internal/alignment` define today, field for field and JSON or YAML tag for tag:

- `parentReviewRound`, with `Number`, `Dir`, `Request *…` (`request.json`: `opened_at`, `state` and the other keys), `Delivery` (`delivery.json`: `delivered_at`, `pings`, the rest), `Verdict *…` (`verdict.json`: `kind`, `recorded_at`, `consumed`, `superseding`) and `Alignment *…` (`alignment.json`: `spawned_at`, `state`, `report`, `reason`), with small unexported struct types for the four files and the state constant `applied`;
- `parseParentReviewRound`, decoding exactly as `parentreview.ParseRoundDir` does: each file decoded only when present, an unknown key tolerated, an undecodable file an error naming it;
- `parentReviewRoundNumber`, accepting `round-<N>` with N a positive decimal, as `parentreview.RoundNumber`;
- `parentReviewDirRel`, the archived round root, `loomengine.LoomReviewsDirRel()` joined with `parent-review`, so `internal/reportengine` no longer reads `loomengine.LoomParentReviewDirRel`, which card 5 deletes;
- `alignmentAttempt` (`attempt.json`, with `number`, `point`, `token`, `spawned_at`, `applied` and the rest, plus `Dir` never serialized) and `alignmentVerdict` (`verdict`, `reason`, `question`, `superseded`), with the verdict word constant `approve`;
- `parseAlignmentAttempt`, decoding as `alignment.ParseAttemptDir` does: `attempt.json` required, the verdict file named `verdict-<token>.yaml` read when present, no verdict validation.

`RunRow`'s `parentRounds` and `alignmentAttempts` take the new types, the two wake readers, `readSegments`, `settlerOf` and `runAttempts` read through them and `parentReviewDirRel`, the test fixtures `parentRoundFiles` and `parentRoundTimed` lay archived rounds out under `parentReviewDirRel()`, and `allowedImports` drops `internal/alignment` and `internal/parentreview`, which the stale-entry check otherwise fails on.
Every report figure stays as it is: the runs, wakes and timing-stops reports' existing tests build archived round and attempt files through `fake_test.go` and `timing_test.go` and keep passing unchanged, which is the coverage for the copied decoders.

**ImpactSummary:** Six reportengine members switch from two external packages to local decoders; report output is unchanged.
