# Card 2 — Go computes each key's open rounds and the circling evidence

**Edit:**
- `internal/shedadapters/bouncerfacts.go`
- `internal/shedadapters/bouncerfacts_test.go`

**Create:**
- `internal/shedadapters#circlingEvidence`
- `internal/shedadapters/circlingevidence_test.go`

**Uses:**
- `internal/shedadapters/bouncerfiles.go`

**Intent:**
D2 defines `CIRCLING` as no progress: a gating finding on a ledger key that is open in the judged round and was open in an earlier round, unchanged or reopened.
Go, not the judge, computes which rounds a key was open in.

In `bouncerfacts.go`:

- Replace the judge-claimed `rounds` union in `recurringKeys` with Go's own reading: for ledgers `1..last`, a key's rounds are the ledger rounds whose entry for that key has `status: open`.
  A key is listed when it was open in two or more distinct rounds; `Reopened` keeps its meaning (resolved in an earlier ledger, open in a later one).
  The facts file's "Recurring keys" section then names, per recurring open key, the rounds it was open in, and its heading line says so.
- Add `circlingEvidence(runDir, round)`: it reads ledger `round` (the one the judge just wrote) and ledgers `1..round-1`, and returns, sorted, every key that in ledger `round` is `open` and gating, and whose open rounds include `round` and at least one earlier round.
  Gating is the judge stencil's existing definition: `class: design` at `MEDIUM` or `BLOCKING`, or any `BLOCKING`, read from ledger `round`'s own `class`/`severity` (Card 1); an unlabelled entry is not gating.
  An unreadable or unparseable ledger contributes no rounds; an unreadable ledger `round` yields no evidence.
  A rising finding count or new keys alone never produce evidence, since only keys open in two rounds count.

The `Gating` column of the facts table keeps its current definition; this card does not change it.

Tests in `circlingevidence_test.go`, over ledgers written into a `t.TempDir()`: a gating key open in rounds 1 and 3 and resolved in 2 is evidence at round 3 (`rounds: [1, 3]`); an unchanged gating key open in 2 and 3 is evidence; a non-gating recurring key is not; new gating keys only at round 3, however many, are not; a key resolved in ledger 3 is not; a missing ledger 3 gives none.
Extend `bouncerfacts_test.go`'s expectations only where the recurring-keys rendering changed.

**ImpactSummary:** The facts file's recurring-keys list now reflects Go-computed open rounds rather than the judge's `rounds` claims; the new function has no caller until Card 9.

**Commit:** 2: Go computes a ledger key's open rounds and the circling evidence
