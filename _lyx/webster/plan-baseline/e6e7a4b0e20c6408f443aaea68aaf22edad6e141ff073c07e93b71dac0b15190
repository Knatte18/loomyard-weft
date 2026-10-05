# Card 1 — Ledger entries carry class and severity

**Edit:**
- `internal/shedadapters/bouncerfiles.go`
- `internal/shedadapters/bouncerfiles_test.go`
- `contracts/stencils/bouncer/bouncer-template-judge.md`

**Intent:**
The circling guard (D2) reads gating from the ledger the judge writes for the judged round, and today a ledger entry carries only `key`, `rounds` and `status`.
Add two optional fields to a ledger entry, `class` and `severity`, to `ledgerEntryYAML` and `ledgerEntry`, and validate them in `parseLedger`:
when present, `class` must be one of the `burlerengine` class constants (`classOrder`) and `severity` one of the `burlerengine` severity constants (`severityOrder`), each with an error naming the entry key;
when absent they stay empty, so every ledger an older judge wrote still parses and an unlabelled entry reads as non-gating.
The exported `LedgerEntry` mirror in `ledgeraccess.go` is left untouched; Card 15 deletes it.

In the judge stencil's ledger section, extend the YAML example and the frontmatter rules: every `status: open` entry carries `class` and `severity`, copied from the latest review's finding the judge maps to that key (after any relabel it names in the rationale), and a resolved entry may omit them.

Tests in `bouncerfiles_test.go`: an entry with valid `class`/`severity` parses and carries them; an out-of-vocabulary class or severity fails naming the key; an entry with neither parses with both empty.

**ImpactSummary:** Additive to the ledger grammar; existing ledgers keep parsing, and only the judge prompt's ledger instructions change.

**Commit:** 1: Ledger entries carry the finding's class and severity
