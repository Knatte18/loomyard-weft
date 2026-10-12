# Lens review

Every fork returned a report.

- criteria-coverage, MEDIUM, a finding leaves the CLI's exit at 0 untested: fixed; card 15's integration run seeds a decision record with a malformed heading and asserts one finding with exit 0.
- criteria-coverage, MEDIUM, a hand landing on a partial tag has no home: fixed; card 13 puts it in `Report.Landings` with its tag and the week's `Hand` count, adds the LLM step only for a full run, and adds a row.
- criteria-coverage, MEDIUM, card 15's trailer can name a commit the prime already holds: fixed; the test makes a new pair code commit before the records commit.
- criteria-coverage, LOW, parent-review rounds counted by request, not review file: rejected; a round's `review.md` exists only on a reject or an approve that carries one (`internal/parentreview/store.go:62`, `ErrEmptyReviewFile`), so counting by review file would drop plain approve rounds, and D9 takes the outcome from the verdict record.
- criteria-coverage, LOW, "no data" for stop classes not pinned as `nil`: fixed in card 14's row.
- criteria-coverage, LOW, flag defaults and `--fresh-days 1` untested: fixed; card 15's integration run uses the default `--since` and `--fresh-days 1` and asserts `selection.since`.
- criteria-coverage, LOW, a verify retry's own paused change counted as paused: fixed in card 7's rule and row (merged with row-reachability's LOW 4).
- criteria-coverage, LOW, the wake mean drops `land-ready-tasks` without a Shared Decision: fixed; added `wake-mean-without-ready-tasks`.
- row-reachability, BLOCKING, `CommitAnchoredPaths` writes no trailer (`internal/fabricengine/commitweftpaths.go:75`): fixed; card 15 commits through `Fabric.Commit`.
- row-reachability, BLOCKING, the hub template's commits are in range and read `unmapped`: fixed; card 15 asserts exactly one landing for the slug and does not count the template's.
- row-reachability, MEDIUM, the trailer sits below the tag and a limited walk misses it: fixed; card 13 walks to the first trailer with no limit, corrects the stated reason, and puts one row's trailer three commits below the tag.
- row-reachability, LOW, the retry's paused change: fixed with criteria-coverage's LOW above.
- row-reachability, LOW, `AccountedChild` has no carrier on the run: fixed; card 11 keeps `Changes` and `AccountedChild` in unexported `RunRow` fields.
- forced-edits, BLOCKING, the sole-parser scan's literal `"history_entries: "` also sits in `internal/loomcli/entryobservation.go:164`: fixed; card 1 scans the title `"loom halted "`, which only `internal/loomcli/halt.go:73` spells.
- forced-edits, LOW, `writeHistoryRows` fate unstated: fixed; card 1 keeps it for the crash-resume note.
- forced-edits, LOW, ImpactSummary claims two callers: fixed; one production caller, `reflectHalt`.
- forced-edits, LOW, the archive prefix still spelled in `remoteArchiveTags` and `hasWeftEvidence`: fixed; card 6 edits both.
- forced-edits, LOW, the `TestLoad` version-2 row already exists (`internal/boardengine/store_test.go:1235`): fixed; card 6 extends the runner to every row with a body, error rows included (merged with test-economy's LOW 4).
- forced-edits, LOW, `internal/friction/doc.go` names the halt note without its new owner: fixed; card 1 edits it.
- test-economy, MEDIUM, pure `ArchiveTagSlug` rows in an integration-tagged test (`internal/fabricengine/archive_integration_test.go:1`): fixed; new untagged `TestArchiveTagSlug`, fabricengine budget 645.
- test-economy, LOW, card 11 repeats card 7's classifier rows: fixed; card 11 keeps the escalation-map row and one wiring row.
- test-economy, LOW, card 14 repeats card 10's ISO-week boundary: fixed; card 14's row asserts bucketing by `RunRow.Week`.
- test-economy, LOW, the `TestLoad` version-2 duplicate: fixed with forced-edits' LOW above.
