# Card 9 — crash-resume-note-evidence

**Edit:**
- `internal/loomengine#EntryObservation`
- `internal/loomcli/entryobservation.go`
- `internal/loomcli/handoffvoucher_test.go`
- `internal/loomcli/halt_test.go`

**Intent:** D7: the `loom-crash-resume` note records why it fired and lists every history entry, so the reader can tell a real crash from a false positive from the note alone.

`EntryObservation` in `internal/loomengine/anomaly.go`:

- Replace `RecentHistory` with `History`, carrying every history entry observed at entry, with its doc comment updated.
- Add the voucher reading beside `Vouched`: `VoucherFound bool`, `VoucherHistoryLength int` and `VoucherState shedengine.State`, the recorded values when a voucher was found.
  `DetectCrashResume` keeps reading `Vouched` only.

`entryobservation.go`:

- `consumeHandoffVoucher` returns the voucher it read (found, history length, state) alongside the match, and `observeEntry` fills the three new fields from it.
- Drop `recentHistoryRows` and the trailing-five cut; `observeEntry` carries the whole history.
- `writeCrashResumeNote` adds, after `history_entries`, `run_lock_held: <true|false>`, then either `voucher: none` or `voucher_history_entries: <n>` and `voucher_state: <state>`, so the recorded values sit beside the observed `history_entries` and `state`; the history section lists every entry through `writeHistoryRows`.

Tests:

- `handoffvoucher_test.go`: `TestObserveEntry_ConsumesHandoffVoucherIntoObservation` asserts the full `History` instead of the last five rows, and the voucher fields for a matching voucher, a mismatching one and none.
- `halt_test.go`: `TestStep_CrashResumeNoteFollowsTheHandoffVoucher`'s `MarkerRemoved` row, which drives two real steps, asserts `run_lock_held: false` and `voucher: none` in the note; add a row whose voucher is replaced by one recording a different history length, asserting the note names `voucher_history_entries` and `voucher_state` beside the observed ones; seed a history longer than five entries in one row and assert every row is listed.

Grep for `recentHistoryRows` and `RecentHistory` before finishing: every use moves to the new field in this card.

**ImpactSummary:** One loomengine struct gains voucher fields and carries full history; its only producer and consumer are loomcli's entry observation and crash-resume note.
