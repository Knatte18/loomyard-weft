---
verdict: APPROVED
---

No findings.
The range is `git log --first-parent --no-merges -p fd539e313..HEAD`, one commit (`1a0e9e54c`), card 16.

## Focus questions

- `readLoomRecords` in `internal/reportengine/runs.go` now calls `shedrun.ParseSeed(seedBytes)`.
  The error branch, its `the seed does not decode` reason and its `Finding{Tag, File: rels.Seed, Error}` are unchanged.
  No file outside `_lyx` still names `ParseSeedTolerant`, docs and `doc.go` comments included.
- `ParseSeed`, `decodeSeed`, `ReadSeed` and `WriteSeed` are untouched.
  `decodeSeed` still has two callers, `ParseSeed` and `WriteSeed`.
- `TestCompute_SelectsRuns`: the `badseed` tag carries `{"recipe":"loom","driver":"rust"}` and no status.
  `ParseSeed` refuses `rust` through `ValidateDriver`, so the tag reaches the seed error branch before any status read.
  The row asserts the tag in `partial` and a finding naming the seed file.
  The `broken` row's partial tag and status-file finding are unchanged.
- `TestParseSeed`: only the per-row and trailing `ParseSeedTolerant` assertions went.
  The `ParseSeed` refusal rows still assert.
  No test of the retired criterion was added.
  No new top-level test was added, so there is no overlap with `TestSeed_UnknownKey` or the report's unknown-keys row, which keeps pinning that an unknown key reads.
- Side-token rule: the added lines spell no fabric side token and no `_lyx` or `.lyx` literal.
  The `badseed` fixture reaches paths through `recordPaths`.

`lyx gate test ./internal/shedrun ./internal/reportengine` passes.
