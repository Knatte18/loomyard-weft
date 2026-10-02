# Card 3 — move-begun-card-hashes

**Edit:**
- `internal/websterengine/fingerprint.go`
- `internal/websterengine/rebaseline.go`
- `internal/websterengine/fingerprint_test.go`
- `internal/websterengine/rebaseline_test.go`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine/recoverbatch_test.go`

**Prosa:**
- `internal/websterengine/doc.go`

**Intent:**
Every sanctioned plan rewrite moves the recorded `CardHashes` of each begun card it changed, in the same restamp that adopts the rewrite (#330, decision D3).

In `fingerprint.go`:
- Split the current body of `restampFingerprint` into `restampBaseline(st *State, planDir, websterDir string) error`, which records the fingerprint, the per-file hashes and the stored baseline exactly as today and touches no batch record.
- Add `moveBegunCardHashes(st *State, before, after map[string]string)`: for every batch record's `CardHashes` entry, with the card file named `<id>.md`, set it to `after[file]` only when it equals `before[file]`.
  A card whose recorded hash differs from `before` keeps its old hash, so a card an earlier untracked rewrite already moved stays refused; a file absent from either map is left alone, as is a state with no `PlanFileHashes`.
- `restampFingerprint` captures `st.PlanFileHashes` before calling `restampBaseline`, then calls `moveBegunCardHashes` with the old and new maps.
  Every sanctioned-rewrite site already funnels through it (begin-batch, `Run` via `restampAndSaveFingerprint`, record-batch and recovery's binding and drift-repair restamps, and validate via `RestampPlanBaseline`), so no caller changes.
  Its doc comment names the hash move and its bound.

In `rebaseline.go`, `Rebaseline` calls `restampBaseline`, never `restampFingerprint`: accepting an operator edit never moves a begun card's hash.
`rebaseline` and the begun-card checks in record-batch and recovery keep comparing exact hashes.

Bound: this moves a begun card's recorded hash without an operator naming it.
It runs only at the sanctioned-rewrite restamps, each of which runs after a foreign-edit check passed in the same call (`PlanEditError`, or run entry's fingerprint compare); no site restamps after a check that failed, so an edit on disk when the call starts is refused, never adopted.
The check runs before the rewrite, not atomically with the restamp: an edit landing between the two in the same call is adopted with the rewrite, and a begun card's hash now moves with it, where today `batchCardEditError` would still refuse it later.
That window is one call's check-to-restamp span: quarry resolution at begin-batch and run entry, and the card-verify re-runs, done-checks and delta at record-batch and recovery.
It moves only a card whose recorded hash equals the pre-rewrite `PlanFileHashes` entry for its file.

Rewrite `doc.go`'s staleness-guard section to state that the restamp moves begun card hashes, the precondition above, and that `rebaseline` never does.

Tests:
- `fingerprint_test.go`: `moveBegunCardHashes` moves a matching hash, keeps a mismatching one, ignores files absent from either map, and is a no-op on empty `PlanFileHashes`.
- `rebaseline_test.go`, #330 Seen: batches 2–4 begun; a later `BeginBatch` canonicalizes a handle that rewrites text in begun cards; the operator then edits pending card 5, and `Rebaseline` naming card 5 succeeds.
  A second case does the same with a record-batch handle binding as the rewrite.
- `rebaseline_test.go`: a foreign edit to begun card 2 followed by a `BeginBatch` call (refused by `PlanEditError`, so no rewrite runs) leaves card 2's recorded hash unchanged, and `Rebaseline` naming card 2 is still refused.
- `recordbatch_test.go` and `recoverbatch_test.go`: a foreign edit to a begun card is refused by `RecordBatch` and by `PersistRecoveryTerminal`, and the refusal leaves `CardHashes` unchanged.
  A second case runs a canonicalizing `planglyph.ValidateDispatch` after the foreign edit with no restamp (validate's path when its edit check refuses), then shows both verbs still refuse.

**ImpactSummary:** every sanctioned-rewrite restamp now also updates begun batches' `CardHashes`; `Rebaseline` switches to a restamp that leaves them alone.
