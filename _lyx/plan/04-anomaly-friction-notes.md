# Card 4 — Route detected anomalies into friction notes

**Edit:**
- `internal/loomengine/anomaly.go`
- `internal/loomengine/anomaly_test.go`
- `internal/loomcli/entryobservation.go`
- `internal/loomcli/arm.go`
- `internal/loomcli/halt.go`
- `internal/loomcli/cli.go`
- `internal/loomcli/entryobservation_test.go`
- `internal/loomcli/handoffvoucher_test.go`
- `internal/loomcli/gotoentry_test.go`
- `internal/loomcli/halt_test.go`
- `internal/loomcli/step_test.go`

**Intent:**
Each Go-detected anomaly becomes a Go-authored friction note the reflection reads (decision D3), written through `friction.NotePath`, only when Tier 2 is on (`c.frictionDir != ""`), and a write failure is warned and never changes the verb's exit or envelope (`PATTERN-friction-capture`).

Halt kinds, `internal/loomcli/halt.go`:

- `haltNote` replaces `HistoryLen int` with `History []shedengine.HistoryEntry` and gains the anomaly kind, derived through `loomengine.ClassifyHalt(n.State, n.Reason)` inside `writeHaltNote`.
- The `loom-halt` note keeps its title line, fenced reason, `producer`, `state`, `history_entries` and `trace_file` lines, and adds an `anomaly: <kind>` line and a history section with one `- <producer> / <outcome> / <at>` row per entry (an explicit "no history entries" line when empty).
- `loomAfterStep`, `loomPostRun` (`internal/loomcli/arm.go`) and `failedHalt` pass the full history they already hold instead of its length.
  One halt still writes one note.

Crash-resume, `internal/loomengine/anomaly.go` and `internal/loomcli/entryobservation.go`:

- Rename `EntryObservation.CleanStepHandoff` to `Vouched`, and add `RecentHistory []shedengine.HistoryEntry`, the last history rows (at most five) observed at entry; update the struct's and `DetectCrashResume`'s comments.
- `observeEntry` fills `RecentHistory` from the status it already reads, and `Vouched` from `consumeHandoffVoucher`.
- Add a crash-resume note writer beside `observeEntry`, writing note id `loom-crash-resume` through `friction.NotePath`: a title line naming the producer, then `slug`, `parent`, `state`, `current_producer`, `history_entries`, `verb` (`run` or `step`), `trace_file` (`logger.TraceFile()`), and the recent history rows in the halt note's row format.
- Replace `observeStepEntry` and `loomPreRun`'s inline observation with one `loomCLI` method taking the verb, which observes and, when `loomengine.DetectCrashResume` reports a crash, writes the note; `loomPreRun` calls it with `run` at its current position, `loomPreStepUnmarked` with `step` at its current last position, so a refused step still neither observes nor consumes the voucher.
  A busy probe (run lock held) stays no anomaly.
- Delete the `entryObservation` field from `loomCLI` (`internal/loomcli/cli.go`): writing at entry replaces the deferred post-run filing, so nothing carries the observation forward.
- The note is not reflected on at once; it waits in the friction directory for the next reflection, as every other note does.

Tests, all untagged:

- `internal/loomengine/anomaly_test.go`: `TestDetectCrashResume` rows use `Vouched`.
- `internal/loomcli/halt_test.go`: `TestWriteHaltNote_NamesProducerAndReason` becomes a table over a budget-prefixed blocked reason, another blocked reason and a failed halt, each asserting one `loom-halt.md` naming its `anomaly:` kind, producer, state, reason, history rows and trace file.
  Add one step-level test over the `handoffSteps` helper card 1 kept, with a real `PreStep` that observes: `MarkerKept` (a completed step then a second step) writes no `loom-crash-resume.md`, and `MarkerRemoved` writes one carrying the fields above.
  The consume-once and voucher-match behavior is already pinned by `TestConsumeHandoffVoucher` and `TestObserveEntry_ConsumesHandoffVoucherIntoObservation` (renamed by card 3), so no new test repeats it.
- `internal/loomcli/entryobservation_test.go`: a table over the no-note cases no existing test covers — run lock held, state not `running`, empty history, Tier 2 off (empty friction directory) — plus a row where the friction directory is unwritable, asserting the verb's hook still returns nil.
- `internal/loomcli/handoffvoucher_test.go` and `internal/loomcli/gotoentry_test.go`: retarget `CleanStepHandoff` to `Vouched` and any comparison with a zero `EntryObservation`, which `RecentHistory` makes non-comparable.
- `internal/loomcli/step_test.go`: the busy-refusal test asserts no `loom-crash-resume.md` was written and the voucher is still on disk, instead of reading the deleted `entryObservation` field.

**ImpactSummary:** halt notes gain the anomaly kind and history rows, and run/step entry writes a `loom-crash-resume` note; no exit code or envelope changes.

**Commit:** 4: loom: write detected anomalies as friction notes
