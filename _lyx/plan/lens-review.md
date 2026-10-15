# Lens review

Every lens fork returned a report.

- criteria-coverage, branch-coverage, row-reachability (merged) — MEDIUM — fixed as proposed: card 1's refused rows gain set-deps adding D to an X at `running · Webster`, refused by `checkRunLocks` after `fn`, asserting `errors.Is(err, ErrRunLocked)` and nil siblings.
- branch-coverage, row-reachability (merged) — MEDIUM — fixed as proposed: card 1's batch-pair row becomes X newly on [D, E] and Z newly on D with E stored, expecting one sibling with `Deps` [D, E], so the both-sides union is reached.
- branch-coverage (merge `set_status` marking X done) — MEDIUM — fixed as proposed: card 1 adds a `MergeTasks` row with `set_status` done on X, expecting no sibling.
- branch-coverage (merge refused by `set_status` on a missing task returns nil siblings) — MEDIUM — disputed, premise false: that refusal is `fn` returning an error inside `internal/boardengine#Store.MergeTasks`, the same early-return branch of `writeReportingSiblings` that card 1's note-`depends_on` and unknown-dependency rows reach, so a row already reaches it.
- branch-coverage (notice says both "newly wait") — LOW — fixed as proposed: card 2's line now reads `"<X>" now waits on <D, ...>, which "<Y>" also waits on, and the two share module labels <L, ...>: …`.
- branch-coverage (more than one pair untested) — LOW — fixed as proposed: card 1 adds a two-sibling row (Y1, Y2) and card 2 a set-deps case asserting two notice lines.
- branch-coverage, row-reachability (merged: run-settings plus sibling notice tested only through `upsert-batch`) — LOW — fixed differently: the proposed `upsert` or `merge` row is not added, because the combination cannot occur there: `prepareRunSettings` (`internal/boardcli/runsettings.go:67-69`) gives a notice only when the upserted entry itself is held past Seed-Child, and `checkRunLocks` (`internal/boardengine/runlock.go:69-86`) refuses that entry's `depends_on` change; card 2's Intent now says the combination is reachable only through `upsert-batch`.
- row-reachability (seeds not stated as tasks) — LOW — fixed as proposed: both cards state that every seeded entry is a `"kind":"task"` carrying `bug`.
- test-economy (single-X `UpsertTasksBatch` row redundant) — LOW — fixed as proposed: the row is dropped, and the batch-pair row covers `UpsertTasksBatch`.
- test-economy (refused rows re-pin the error text and rollback) — LOW — fixed as proposed: the refused rows assert only the error and nil siblings, naming the existing tests that pin the text and rollback.
- test-economy (CLI refused case reaches nothing new) — LOW — fixed as proposed: card 2 drops the refused case and says why.
- forced-edits (step's test file unnamed) — LOW — fixed as proposed: card 2 places `stepSiblingWarning` in `internal/boardcli/cli_test.go`.
- forced-edits (`board_test.go` header falls out of date) — LOW — fixed differently: the header is not reworded, since `redundant-file-target` refuses `internal/boardengine/board_test.go` beside card 1's `TestPriorityRoundTrip` and `TestEntryDates` members; `TestWriteSiblings` moves instead to a new `internal/boardengine/siblings_test.go`, listed under `Create`.
- forced-edits (`//testtiming:keep` reason on `TestHelpSchema_LeafCommands`) — LOW — fixed as proposed: card 2 updates the reason to include the sibling-warning sentence.
- criteria-coverage (new top-level test, reason not stated) — LOW — fixed as proposed: card 1 says the Store-level tests cannot see the siblings, which only the `Board` methods return.
