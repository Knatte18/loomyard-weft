# Card 12 — report-timing-verb

**Create:**
- `internal/reportcli#defaultTimingDays`
- `internal/reportcli#timingCommand`
- `internal/reportcli#parseTimingOptions`
- `internal/reportcli#resolvePrime`
- `internal/reportcli#livePairs`
- `internal/reportcli#journalSleeps`
- `internal/reportcli#sleepsFromJournal`
- `internal/reportcli#TestTimingSleepNote`
- `internal/reportcli#TestParseTimingOptions`
- `internal/reportcli/timing.go`

**Rename:**
- `internal/reportcli#TestConvergence_RefusesBadFlags` -> `internal/reportcli#TestReportVerbs_RefuseBadFlags`
- `internal/reportcli#TestConvergence_RefusesOutsideThePrimeThenReportsTheSeededRun` -> `internal/reportcli#TestReportVerbs_RefuseOutsideThePrimeThenReportTheSeededRun`

**Edit:**
- `internal/reportcli#Command`
- `internal/reportcli#convergenceCommand`
- `internal/reportcli#refuseNonPrime` -> `func refuseNonPrime(verb, worktreeName, primeName string, primeNameErr error) error`
- `internal/reportcli#TestRefuseNonPrime`
- `internal/reportcli/doc.go`
- `docs/overview.md`
- `tools/sandbox/SANDBOX-CORE-SUITE.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add the verb `lyx report timing [--since <date>] [--by day|week] [--format json|markdown]` beside `convergence`: prime-only, read-only, taking no lock and making no commit.

Flags (`parseTimingOptions`), checked before any cwd resolution, each refusal a JSON error with exit 1 naming the flag, the value and the accepted form, as `convergence`'s are:
`--since` is a UTC `YYYY-MM-DD` date defaulting to `defaultTimingDays` days before today's UTC date; a date after today is accepted and yields an empty report;
`--by` is `day` (the default) or `week`, `reportengine.BucketDay` or `BucketWeek`, refused otherwise as "report: --by \"<value>\" is not a bucket: expected day or week";
`--format` is `json` (the default) or `markdown`.
`TestParseTimingOptions` in `internal/reportcli/cli_test.go` pins the defaults against a fixed `now` of 2026-10-15T23:30:00Z: no flags give `Since` 2026-10-08T00:00:00Z and `By` `day`; `--since 2999-01-01` is accepted as that date; `--by week` gives `BucketWeek`.

Prime and geometry.
`resolvePrime` holds what `convergenceCommand` does today after its flags: the cwd from the context, `lyxcwd.Resolve`, `fabricengine.RequireDrivableWorktree` and the prime-name check, each refusal naming `lyx report <verb>`; both verbs call it, then `primeGeometry`.
`refuseNonPrime` takes the verb, so its refusal names `lyx report timing` or `lyx report convergence`, and `TestRefuseNonPrime`'s rows pass a verb and assert it in the message; `internal/reportcli/refusal.go`'s file comment says the rule serves both reports.
`livePairs` lists the hub's code worktrees through `fabricengine.CodeWorktrees(location.WorktreePath())` and returns the base names of every worktree but the main one, the slugs whose pair is live; the engine reads a pair branch whose slug is not among them as abandoned.

Sleep.
`journalSleeps` runs `journalctl` with `sleepjournal.Args()` once per invocation, after `exec.LookPath` finds it, logging the spawn and its completion at Info through `internal/logger` (`PATTERN-spawn-observability`); a `LookPath` failure is handed on as the run error without a spawn.
`sleepsFromJournal` calls `sleepjournal.Read` (card 2), converts its intervals to `reportengine.Interval` and returns the note the output's `sleep_note` carries: with a clause, "No machine sleep is subtracted: <clause>."; with sleeps, "Machine sleep, read from the systemd journal, is left out of every interval."
`TestTimingSleepNote` in `internal/reportcli/cli_test.go` covers the note for a run error wrapping `exec.ErrNotFound` (naming `journalctl is not on PATH`), a failing run, and a recorded sleep, the last also asserting the converted interval; `sleepjournal`'s own test pins the remaining clauses.

Output.
`timingCommand` opens the sources through `reportengine.Open`, calls `reportengine.ComputeTiming` with the parsed options, the live pairs and the sleeps, and prints either `{"ok":true}` with `selection`, `runs`, `phases`, `buckets`, `sleep_note` and `findings`, or `reportengine.RenderTimingMarkdown`'s document; both exit 0 whatever findings hold, and an error reading the repositories themselves is a JSON error with exit 1.
`Command` adds the verb, and its group text names both reports.

Tests.
The two tests that now cover both verbs are renamed per the plan's rename mechanic, then changed.
`TestReportVerbs_RefuseBadFlags` carries the verb in each row and gains timing rows: `--since 2026-13-01`, `--by month` naming `--by`, `month`, `day` and `week`, and `--format yaml`, each with exit 1 before any cwd resolution.
`TestReportVerbs_RefuseOutsideThePrimeThenReportTheSeededRun` stamps its seeded run inside the default window: the seed's `started_at`, the batten history entries' and the loom `Finalize` entry's `at` are set an hour apart, ending an hour before the test's start, so card 8's span opens at the batten record's first entry and overlaps the window.
After the seeded run is tagged it asserts: `timing` from the pair's worktree refused naming the prime; `timing` from the prime holding a landed run whose `tag` is the seeded run's tag, with a `phases` object, and a `sleep_note`; no finding naming the template pair's records branch as abandoned, since `livePairs` names that live pair, so a wrong slug derivation fails the row; `timing --since 2999-01-01` printing no run and no bucket; and `timing --format markdown` printing the four timing headings.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/reportcli` to 5.

Docs.
`internal/reportcli/doc.go` documents the verb, its flags and refusals, the live-pair listing, the journal read and the output keys.
`docs/overview.md`'s `internal/reportcli/` and `internal/reportengine/` lines name the timing report beside convergence.
`tools/sandbox/SANDBOX-CORE-SUITE.md`'s S13 scenario gains Watch lines: `lyx report timing` from the prime prints one `{"ok":true}` envelope holding `runs`, `phases`, `buckets` and `sleep_note`; `--format markdown` prints its four section headings; `--by month` refuses naming the flag; and nothing on disk or in history changes.

**ImpactSummary:** refuseNonPrime's one production caller moves into resolvePrime on this card and convergence keeps its flags, output and refusals with only the verb named in its prime refusal; the new verb reads only.

**Commit:** 12: Add lyx report timing
