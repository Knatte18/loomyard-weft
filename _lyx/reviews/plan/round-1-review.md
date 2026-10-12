---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/16-single-seed-parser.md:23
    summary: The attack-surface bound says the shut-out seeds are ones WriteSeed could never have written, but WriteSeed does not refuse an empty params key
    fix: In card 16's Intent, replace the sentence at line 23 with "That is the only behavior the change lets through or shuts out, and it is bounded to tags whose seed `ReadSeed` already refuses, so their run never bootstrapped and never reached Finalize; no lyx writer produces such a seed, since `WriteSeed` refuses an unknown driver and `lyx shed seed` refuses an empty `--param` key."
---

### [LOW] The stated bound misnames what keeps an empty params key out

**Location:** `_lyx/plan/16-single-seed-parser.md:23`

**Issue:** The Intent bounds the newly shut-out seeds to "tags whose seed `WriteSeed` could never have written".
That holds for the driver half: `WriteSeed` calls `ValidateDriver` (`internal/shedrun/seed.go:181`), and the driver vocabulary has been closed at `go` and `llm` since `5288e0f30`.
It does not hold for the params half: `WriteSeed` validates run id, recipe and driver only, and never checks for an empty params key (`internal/shedrun/seed.go:174-221`).
The empty key is refused upstream by `parseSeedParams` in `internal/shedcli/seed.go:51`, and at read time by `ReadSeed` through `ParseSeed`.
The bound itself is sound, since any seed `ParseSeed` refuses was already refused by the live `ReadSeed`, so its run never bootstrapped; only the sentence naming the reason is wrong.

## Focus settlements

- **Fidelity.**
  The decision record's `Added after Discussion (parent, 2026-10-12)` entry says convergence-plan "drops ParseSeedTolerant and its 'ReadSeed still rejects it' criterion, uses the tolerant ParseSeed".
  That supersedes D5's sentence that the report never calls `ParseSeed` for the seed, and the acceptance criterion "`shedrun.ReadSeed` still rejects it".
  D5's general rule stays: other records keep their tolerant parse-from-bytes functions, and the report's seed parse still tolerates unknown fields, which the existing "a seed and a status carrying unknown keys read" row pins.
  The Constraint "no reader a live run or gate calls changes" holds, because the card leaves `ParseSeed`, `decodeSeed`, `ReadSeed` and `WriteSeed` unchanged.
  The entry's "keeps `internal/alignment/records.go` compiling" clause is carried: the import is present, and the card leaves it alone.
  Findings F1's fix list maps one to one onto the card, and the card adds no scope beyond it.
  No existing test pins "`ReadSeed` rejects an unknown field": `TestSeed_UnknownKey` (`internal/shedrun/seed_test.go:201`) pins the opposite.
- **Reachability.**
  `selectRuns` filters only on `--since` (zero in this test) and on the tag naming a slug, which `badseed` does.
  `readLoomRecords` reads the seed and calls the parser before it checks the recipe and before it reads the status.
  `{"recipe":"loom","driver":"rust"}` decodes, then fails `ValidateDriver` inside `ParseSeed`, so it reaches the `the seed does not decode` branch (`internal/reportengine/runs.go:228-231`).
  The fixture's `tagOver` writes seed text verbatim, not through `WriteSeed`, so no fixture-side validation blocks it.
  A missing status file is never read on that path.
- **Attack surface.**
  The only acceptance change is `ParseSeed`'s driver and params-key validation; decoding is the same `decodeSeed` in both parsers.
  The bound is correct in effect, but the sentence naming it is inaccurate: see F1.
- **Member glyphs and test economy.**
  `readLoomRecords`, `TestParseSeed` and `TestCompute_SelectsRuns` are the members changed, each named as a glyph.
  None is re-signed, so no arrow is needed.
  `ParseSeedTolerant`'s callers are only `readLoomRecords` and `TestParseSeed`, both on the card.
  No existing report test reaches the seed error branch: the only undecodable record in `TestCompute_SelectsRuns` is the `broken` status.
  The `badseed` row is a new row in an existing test, not a new test function.
- **Verify coverage.**
  `go test ./...` and `go test -tags integration ./...` cover `internal/shedrun` and `internal/reportengine`.
  The order matches the template: build, vet under integration, tmux and llm, then the untagged and integration test tiers.
