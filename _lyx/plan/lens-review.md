# Lens review

Every fork returned a report.

- test-economy, row-reachability, forced-edits (merged), BLOCKING, card 4: `TestGuardedPickerClose`'s interrupt rows lost the picker capture to `requireReadyAgentPane`. Fixed: the interrupt ops script `["REWIND-PICKER", "REWIND-PICKER", "❯ "]`.
- test-economy, LOW, card 4: the "no picker on an interrupt" row repeats a guarded-interrupt row. Rejected: it is the negative on a picker-capable engine, where the close must send nothing.
- test-economy, LOW, card 4: `TestRun_Interrupt_PlaysEscape` keeps its name. Rejected: a rename adds churn, and the card updates its doc comment.
- test-economy, LOW, card 2: `TestInterruptGuardReadings` overlaps `TestIdleSession` on the needle. Rejected: `RunningTurn` and `RewindPicker` are new public methods, and the picker rows are new.
- criteria-coverage, MEDIUM, card 8: `shuttlePair`'s readings were untested. Fixed: a pure `readTurn` feeds `Busy` and `Background`, pinned by the new `TestShuttlePairReadings`, with the `pausecli` budget raised.
- criteria-coverage, MEDIUM, card 8: no row pinned the settle after the second key. Fixed: `onPaneRead` stamps the pane read, and the still-busy row asserts its gap.
- criteria-coverage, MEDIUM: the by-hand criterion was carried by no card. Fixed: card 8 names it as the operator's live check after landing.
- criteria-coverage, LOW, card 9: no row covered the Info log on a refused second key. Fixed: row added.
- criteria-coverage, LOW, card 8: no row had a picker on the second key. Fixed: row added.
- criteria-coverage, LOW, card 6: the operator-hold return site for an engine without `SessionCycler` has no `wantReason` row. Rejected: it builds the same `SessionBusyError` as the cycler site, and the type assertion covers it.
- criteria-coverage, LOW, card 2: the picker needle is the subtitle plus a no-box condition. Rejected: the subtitle belongs to the picker's heading block, and the no-box condition keeps a quoted subtitle from drawing an Esc, within D8's bound.
- row-reachability, MEDIUM, card 4: `Runner.PaneRunningTurn` and `Runner.InterruptSettle` were untested. Fixed: the `viaRunner` rows of `TestRun_Interrupt_PlaysEscape` assert both, and the capture-failure row runs for the Runner too.
- row-reachability, LOW, card 8: three new rows left the busy script unset. Fixed: busy `[true]` added.
- row-reachability, LOW, card 8: the pane-read counter field was not listed. Fixed: `paneReads` and `onPaneRead` are listed on `fakePair`.
- forced-edits, LOW, card 8: `pausecli` `Short` help was stale. Fixed.
- forced-edits, LOW, card 9: the `pauseTick` godoc was stale. Fixed: `Watcher.pauseTick` is added to Edit.
- forced-edits, LOW, card 9: `fakeSession`'s comment. Rejected: the card's Intent already states the new fields.
