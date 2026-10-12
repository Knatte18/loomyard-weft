# Card 16 — single-seed-parser

**Delete:**
- `internal/shedrun#ParseSeedTolerant`

**Edit:**
- `internal/reportengine#readLoomRecords`
- `internal/shedrun#TestParseSeed`
- `internal/reportengine#TestCompute_SelectsRuns`

**Uses:**
- `internal/shedrun#ParseSeed`
- `internal/shedrun#TestSeed_UnknownKey`

**Intent:**
The decision record's entry `Added after Discussion (parent, 2026-10-12): Seed parsing follows run-robustness D4` drops `ParseSeedTolerant` once run-robustness is merged in, because two parsers for one record contradict `PATTERN-tolerant-records`.
The Merge-In at `7af4bf32f` brought run-robustness's tolerant `ParseSeed`, which accepts an unknown key, yet `ParseSeedTolerant` survived and the report still calls it.

Delete `ParseSeedTolerant` from `internal/shedrun/seed.go`.
`ParseSeed`, `decodeSeed`, `ReadSeed` and `WriteSeed` stay unchanged.
In `readLoomRecords` (`internal/reportengine/runs.go`), read the seed through `shedrun.ParseSeed(seedBytes)` instead; the error branch, its `the seed does not decode` reason and its finding naming the seed file stay as they are.
The report thereby inherits `ParseSeed`'s driver and params validation: a tagged seed naming an unknown driver or an empty params key now reads as a partial run with a finding, where `ParseSeedTolerant` read it as a loom seed.
That is the only behavior the change lets through or shuts out, and it is bounded to tags whose seed `ReadSeed` already refuses, so their run never bootstrapped and never reached Finalize;
no lyx writer produces such a seed, since `WriteSeed` refuses an unknown driver and `lyx shed seed` refuses an empty `--param` key.

In `internal/shedrun/seed_test.go`, `TestParseSeed` drops its `ParseSeedTolerant` assertions: the per-row tolerant check and the `[1]` decode-error check after the loop.
Its rows keep asserting that `ParseSeed` refuses an unknown driver and an empty params key.
No test pins the retired criterion that `ReadSeed` still rejects a seed with an unknown field; `TestSeed_UnknownKey` pins the opposite and stays.
`internal/alignment/records.go` keeps its `encoding/json` import, which `ParseAttemptDir` needs, and this card does not touch it.

Tests:
- `TestCompute_SelectsRuns`'s row "a status that does not decode is a finding and the tag is partial" becomes "a seed or a status that does not decode is a finding and the tag is partial", and adds a second tag, `badseed`, whose seed is `{"recipe":"loom","driver":"rust"}` with no status.
  It expects `badseed`'s tag in `partial` beside `broken`'s, and a finding naming `badseed`'s tag and `recordPaths(w.t, "badseed").Seed`.
  This reaches the `ParseSeed` error branch in `readLoomRecords`, which `ParseSeedTolerant` never took for that seed.
- The existing row "a seed and a status carrying unknown keys read and the run is full" keeps pinning that a seed with an unknown key reads through the report, now through `ParseSeed`.

**ImpactSummary:** One unused-elsewhere shedrun export goes, the report's one seed read switches to `ParseSeed`, and two test functions change; no live reader's acceptance moves.

**Commit:** `16: report reads seeds through ParseSeed; drop ParseSeedTolerant`
