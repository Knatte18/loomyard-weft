---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/shedrun/seed.go:146
    summary: ParseSeedTolerant repeats decodeSeed's body since the parent made decodeSeed tolerant
    fix: 'In internal/shedrun/seed.go, make ParseSeedTolerant call decodeSeed(data) in place of its own json.Unmarshal into a Seed, keeping the "decode: %w" wrap and the DriverGo default for an empty Driver, so the one tolerant decode has one body.'
---

### [LOW] ParseSeedTolerant repeats decodeSeed's body since the parent made decodeSeed tolerant

**Location:** `internal/shedrun/seed.go:146`
**Issue:** Before the merge `decodeSeed` decoded strictly, so the task's `ParseSeedTolerant` needed its own `json.Unmarshal`.
The parent's tolerant-records change made `decodeSeed` a plain `json.Unmarshal` that accepts an unknown key, so the two bodies are now identical and the only difference between `ParseSeed` and `ParseSeedTolerant` is the driver and params validation.
The second parser is still needed, because the report must count a seed whose driver or params `ParseSeed` refuses, and it breaks no invariant.
Only the duplicated decode is left over, and the docs of the two parsers still read correctly.

## Rejected

- `decodeSeed`'s doc comment still claiming it rejects an unknown key: it now says it accepts a key `Seed` does not declare, and `ReadSeed` and `ParseSeed` say the same.
- `ParseStatusTolerant` duplicating or contradicting the parent's read path: the live read goes through `state.ReadJSONNoMkdir` and still refuses an invalid `State` at `run.go` and `goto.go`.
  `ParseStatusTolerant` serves bytes read from a git tree, so the two do not overlap, and `status.go`'s comment on the read gate stays true.
- `PATTERN-tolerant-records` against the report readers: `reportengine` calls no `DisallowUnknownFields`, reads seed and status through the tolerant parses, and decodes the alignment verdict with no `KnownFields`.
  `cmd/lyx/tolerantrecords_test.go` walks every production file, so the new package is covered with no per-package row.
- `PATTERN-leaf-packages`, `PATTERN-sole-parsers` and `PATTERN-cli-cobra`: the parent's `pauserecord` and `pausecli` text and the task's `haltnote` text sit in separate sections and read coherently.
  `reportengine` is listed in `PATTERN-told-geometry`, and `reportcli` has its own engine package, so it needs no cobra deviation.
- `main.go`, `seamsignature_test.go` and the budget: `report`, `pause` and `resume` are all mounted and named in the module list, and both `pausecli` and `reportcli` appear in both seam-signature lists.
- `docs/overview.md` and the sandbox suite: no removed webster pause verb is named, `lyx report` has one module entry, and sandbox scenarios S12 (pause) and S13 (report) are numbered once each, with matching result lines.
- The report's counts against the parent's pause flow: stops, wakes and paused changes come from batten's recorded `child <from> → <to>` history entries, which the parent's pause hold leaves in place.
  The report reads no retired field, and `pause_after` is not read.
- The parent's `attempt.lock` beside `attempt.json`: `ParseAttemptDir` looks up `attempt.json` and the token-named verdict by name, so an extra file in the attempt directory changes no count.
- Test economy across the merge: the parent's `TestSeed_UnknownKey` covers `ReadSeed` over an unknown key and the task's `TestParseSeed` rows cover `ParseSeedTolerant`, so neither repeats the other.
