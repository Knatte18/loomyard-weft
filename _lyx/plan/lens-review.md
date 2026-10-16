# Lens review

Every lens fork returned a report.
Duplicates across lenses are merged into one line, naming each lens.

- criteria-coverage, branch-coverage, row-reachability — MEDIUM — `--full` is wired through cobra with no row reaching it — fixed differently: the proposed `TestHelp` row would pin only the flag's registration, and the proposed separate `RunCLI` row would be a second test of the same output; instead every row of the list test in card 1 now runs the `list` command `intakeCommand` builds, with `SetArgs` under `clihelp.RunRootCtx` (`internal/clihelp#RunRootCtx`), so the compact and `--full` rows reach `intakeList` through the flag.
- criteria-coverage — MEDIUM — the `--full` row never asserts `labels` — fixed as proposed: card 1's full row asserts issue 6's `labels` equal `[bug]`.
- branch-coverage — LOW — the `RecordedIssues` error exit card 1 names has no row — fixed as proposed: card 1 adds a board-load-failure row that writes `{` to board.json after the seed.
- branch-coverage — LOW — card 2's nil-to-empty `labels` branch is unreached — fixed differently: no row can reach it, since the board refuses an entry with no type label (`internal/boardengine/store.go:716`, `internal/boardengine/intake.go:182-184`), so card 2 drops the branch rather than gaining a row, and says why `labels` is never empty.
- branch-coverage — LOW — the close-failure envelope is tested only for a new note, not a fold — disputed, premise false: a note and a fold share one close path in `internal/boardcli#intakeImport` (`internal/boardcli/intake.go:275-283`), and card 2 keeps it as one result map, so there is no fold-only branch for a row to reach.
- test-economy — LOW — the list-failure row runs once per mode but the fetch fails before the mode is read — fixed as proposed: card 1 keeps one list-failure row, in the default mode.
- test-economy, forced-edits, row-reachability — LOW — `TestIntakeImport_CloseFailureReportsEntryAndClosePayload`'s name contradicts its new no-`entry` assertion — fixed as proposed: card 2 renames it to `TestIntakeImport_CloseFailureReportsShortResultAndClosePayload` through a `Rename:` pair, and the overview gains the `## Rename mechanic` section.
- forced-edits — LOW — `intakeIssueView`'s doc comment still says it is what `list` prints — fixed as proposed: card 1 moves `internal/boardcli#intakeIssueView` from `Uses:` to `Edit:` and says its comment names the `--full` view.
- row-reachability — LOW — the empty-inbox row leaves the recorded issue's number open — fixed as proposed: card 1 names the recorded issue 4 under the shared `kept` seed.
- row-reachability — LOW — a table keyed by `full` cannot carry rows that need their own inboxes — fixed as proposed: card 1's rows each carry their own `fakeIssue` list, `failList` setting, arguments and expected outcome.
- row-reachability — LOW — `issue` assertions on a decoded envelope compare against an int — fixed as proposed: card 2 states each `issue` assertion compares against a `float64` (`7.0`, `8.0`, `16.0`).
