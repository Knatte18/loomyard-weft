---
round: 1
focus:
  - "internal/reportengine/runs.go readLoomRecords: confirm the seed read goes through shedrun.ParseSeed and the error branch, its reason string and its finding (tag, seed file, error) are unchanged from before; settle whether any other call site in the repo, including docs and doc.go comments, still names ParseSeedTolerant."
  - "internal/shedrun/seed.go: confirm ParseSeed, decodeSeed, ReadSeed and WriteSeed are untouched, and that decodeSeed still has a caller after the deletion."
  - "internal/reportengine/runs_test.go TestCompute_SelectsRuns: settle whether the new badseed tag actually reaches the ParseSeed error branch (driver validation) and asserts both the partial tag and the finding naming the seed file, and that the broken-status row still asserts what it did."
  - "internal/shedrun/seed_test.go TestParseSeed: confirm only the ParseSeedTolerant assertions went, the ParseSeed refusal rows still assert, and no test of a retired criterion was left or added; check test economy against TestSeed_UnknownKey and the existing unknown-keys row."
  - "Side-token rule: settle whether the diff, including new test fixtures and comments in internal/reportengine, spells a fabric side token or a _lyx or .lyx literal."
---

Single card (16) in a rework generation: the diff is small, so the questions above are about residual references and whether the new test reaches the branch the card names.
