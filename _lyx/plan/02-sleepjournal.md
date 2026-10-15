# Card 2 — sleepjournal

**Create:**
- `internal/sleepjournal#Interval`
- `internal/sleepjournal#Args`
- `internal/sleepjournal#Parse`
- `internal/sleepjournal#Read`
- `internal/sleepjournal#TestParse`
- `internal/sleepjournal#TestLeafInvariant_StandardLibraryOnly`
- `tools/tokencount#TestSleepsFromJournal`
- `internal/sleepjournal/doc.go`
- `internal/sleepjournal/testmain_test.go`

**Delete:**
- `tools/tokencount#sleepEntered`
- `tools/tokencount#sleepReturned`
- `tools/tokencount#journalEntry`
- `tools/tokencount#parseSleepJournal`
- `tools/tokencount#sleepOperation`
- `tools/tokencount#TestParseSleepJournal`

**Edit:**
- `tools/tokencount#journalSleeps`
- `tools/tokencount#sleepsFromJournal`
- `tools/tokencount/main.go`
- `pattern/PATTERN-leaf-packages.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Move the systemd-journal sleep parsing out of `tools/tokencount/sleep.go` into a new stdlib-only leaf `internal/sleepjournal`, so `reportcli` (card 12) and `tokencount` read sleeps, and decide why none were read, through one parser.

`internal/sleepjournal/sleepjournal.go` holds `Interval` (`From`, `To time.Time`), the two message prefixes and the journal-entry shape moved unexported, `Parse` (the body of `parseSleepJournal`, with `sleepOperation` as its unexported helper, unchanged in behavior: a sleep runs from a start entry to the next return entry of the same operation in the same boot, every other line adds nothing, and an undecodable line or timestamp is an error), and `Args`, the argument list after the executable, `-t systemd-sleep -o json --output-fields=MESSAGE,_BOOT_ID --no-pager`, so both callers run the same query.
`Read` maps the output and exit error of one `journalctl` run to the sleeps and a reason clause: with a sleep found the clause is empty; otherwise there are no sleeps and the clause says why nothing is left out, `journalctl is not on PATH` for a run error wrapping `exec.ErrNotFound`, `journalctl failed: <error>` for any other run error, the parse error's text for output that does not parse, and `the journal records no systemd-sleep operation` for output holding none.
These are the clauses `tokencount`'s notes carry today, so its notes are unchanged.
The package spawns no process; each caller runs `journalctl` itself and wraps the clause in its own note.
`internal/sleepjournal/doc.go` says so and names the stdlib-only import set; `leaf_enforcement_test.go` holds `TestLeafInvariant_StandardLibraryOnly` calling `scankit.AssertImportAllowlist(t, "internal/sleepjournal")`; `testmain_test.go` runs the tests under `tmuxkit.Main`.
`internal/sleepjournal/sleepjournal_test.go` holds `TestParse`: the parse rows of `TestParseSleepJournal` moved verbatim, comparing `Interval` values, and the `Read` rows for each clause: a run error wrapping `exec.ErrNotFound`, a failing run, an undecodable line, an empty journal and a recorded sleep (empty clause, one interval).

In `tokencount`, `journalSleeps` runs `journalctl` with `sleepjournal.Args()`, and `sleepsFromJournal` calls `sleepjournal.Read`, converts each `sleepjournal.Interval` to tokencount's own `Interval` and wraps the clause in its existing sentences; every note text is unchanged, so the subtraction and the report header read exactly as before.
`TestParseSleepJournal` is replaced by `TestSleepsFromJournal` in `tools/tokencount/sleep_test.go`, holding the note rows of the old test unchanged, so a changed note fails it, its recorded-sleep row also asserting the converted interval's ends.
The package doc in `tools/tokencount/main.go` names `internal/sleepjournal` as the parser of the journal entries it describes.
`pattern/PATTERN-leaf-packages.md` gains an `internal/sleepjournal` section: imports only the standard library, and spawns no process.
`docs/overview.md`'s package tree gains an `internal/sleepjournal/` line.
`cmd/lyx/testdata/test-budget.yaml` gains `internal/sleepjournal: 2`; `tools/tokencount` keeps 17, one test replacing another.

**ImpactSummary:** tokencount's sleep reading changes its parser's home only, with identical intervals and notes; no other package reads the journal yet.

**Commit:** 2: Move the systemd-journal sleep parser into internal/sleepjournal
