# Card 6 — rebaseline-terminal-batch

**Edit:**
- `internal/websterengine/rebaseline.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/rebaseline.go`
- `contracts/specs/refusal-spec.md`
- `internal/websterengine/rebaseline_test.go`
- `internal/webstercli/verbs_test.go`

**Intent:**
Implement decision D6 (#403): `lyx webster rebaseline --card NN` accepts an edit to a card of a failed, dead or stuck batch, so a one-card fix no longer needs `reset --to start` plus `run --fresh`.

In `Rebaseline`'s begun-batch loop, a begun card whose content hash differs from the one recorded at begin is accepted when all of these hold: the batch's record is `Terminal` with `Status` `failed`, `dead` or `stuck`; `deps.Cards` names that card; the record lists no `Uncheckable` entries; and the batch's recorded card-ID set still equals the plan's (the existing check, unchanged).
Accepting it restamps that card's entry in the batch's `CardHashes` and keeps every other field of the record (StartSHA, warnings, transcripts, suspect paths).
Restamps are applied only once every check has passed, so a refusal leaves state.json untouched as today.
The refusals, each wrapping `ErrRebaselineCardSetChanged` and saying why `--card` cannot accept it:
a `done` batch — its work has landed, so the way forward is the existing "restore those cards in the plan, or" `freshRestartSteps`;
a begun, non-terminal batch — a fork may be in flight, so the way forward names `lyx webster record-batch NN` or `lyx webster recover-batch NN` first, then re-run the rebaseline;
a failed batch whose record lists uncheckable entries — `recover-batch` would refuse it, so the way forward is the same `freshRestartSteps` that `recover-batch`'s own refusal names;
a changed begun card not named with `--card` keeps the existing unnamed-card refusal.
`00-overview.md` stays never rebaselined.

Bound, stated in `Rebaseline`'s doc comment: only cards named with `--card`, only in batches terminal `failed`, `dead` or `stuck`, never a failed batch with uncheckable entries, never a change to a batch's card-ID set, never `00-overview.md`.
What it lets through: for a dead batch the restamp lands while its strand, kept alive when classified dead, may still work on the old card; the restamp stops no strand, and `recover-batch`, the next step, stops that strand before it spawns and archives a late report from it.

Docs: in the refusal-spec, reword the "rebaseline card set changed" row's trigger so an edited card of a failed, dead or stuck batch is no longer in it, and add rows "rebaseline: begun card of a done batch", "rebaseline: begun card of an unfinished batch" and "rebaseline: failed batch recovery cannot check", each with the way forward above; update `doc.go`'s rebaseline paragraph and the `rebaseline` `Long` text in `internal/webstercli/rebaseline.go`.

Tests:
in `rebaseline_test.go`, a table over the batch record's state with one named edited card: `failed`, `dead` and `stuck` accept it, restamp that card's hash and keep StartSHA and warnings; begun non-terminal and `failed` with an uncheckable entry each refuse with their reason and way forward and leave the state unchanged; an unnamed edited card of a failed batch still refuses as unnamed.
The done-batch refusal is already driven by `TestRebaseline_RefusesChangedBegunCardBody` and `TestRebaseline_ForeignEditToBegunCardStaysRefused`, so the table has no `done` row: the first test is extended to assert the new done-batch reason beside its existing `changed since it was begun` substring, which the reworded refusal keeps, and the second keeps pinning the unnamed case.
Adjust either fixture if it used a failed batch.
A dead batch's live strand being stopped before the recovery spawns is already pinned by `TestRecoverBatch_FirstCallSpawnsArchivesStaleReportAndStopsLiveStrand`, so no `recoverbatch_test.go` test is added.
In `internal/webstercli/verbs_test.go`, drive `rebaseline --card NN` through the cobra command over a failed batch: exit 0 and the batch record keeps its start SHA; then `recover-batch NN` through its command spawns with the edited card in its prompt; and `rebaseline --card NN` over a done batch exits 1 with D6's reason.

**ImpactSummary:** Rebaseline accepts a named card edit inside a terminal failed, dead or stuck batch by restamping that card's hash; done, unfinished and uncheckable batches keep refusing with clearer reasons.
