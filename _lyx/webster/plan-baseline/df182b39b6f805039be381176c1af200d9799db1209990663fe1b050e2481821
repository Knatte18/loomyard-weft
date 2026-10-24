# Card 10 — gate-report

**Create:**
- `plan:internal/gateslot#ReadRunRecords` -> `func ReadRunRecords(dir string) ([]RunRecord, error)`
- `plan:internal/gatecli#reportCmd` -> `func reportCmd() *cobra.Command`
- `plan:internal/gatecli#runReport` -> `func runReport(out io.Writer, records []gateslot.RunRecord, slug string, since time.Time) int`
- `plan:internal/gatecli#TestGateReport` -> `func TestGateReport(t *testing.T)`

**Extends:**
- `internal/gatecli#newCommand`
- `internal/gatecli/doc.go`
- `internal/gateslot/doc.go`
- `docs/overview.md`

**Intent:**
Add `lyx gate report [--slug <slug>] [--since <date>]` (D11), so a loom or darn run's gate time is measurable per gate key.

`ReadRunRecords` reads every `.json` record under a run log directory tolerantly, skipping and logging a file that does not decode; a directory that does not exist yet, as on a machine that has run no gate, reads as no records.
`reportCmd` resolves the hub as `lyx gate status` does and reads this machine's records from the board's gate run directory, or the worktree's own outside every hub; `--slug` keeps one slug's records and `--since` (a `YYYY-MM-DD` date) drops older ones; a malformed date is refused naming the expected form, parsed before hub resolution, so the refusal needs no hub.
`runReport` prints an ok envelope with one row per slug and gate key: the tier runs, the carried tiers, the summed slot wait (start minus wait start), the summed wall time, and the launch time with its share, the launch time being the sum of the records' launch gaps and its share that sum over their summed package time.
A record from before `--gate` has no key and groups under its tier list and scope form, such as `tiers 0,1 all`; a record without a launch list adds nothing to the launch time, and a group whose records hold no launch list shows no launch time.
The verb carries the operator audience annotation and changes nothing.

`TestGateReport` feeds `runReport` records old and new, keyed and unkeyed, carried and run, with and without launch lists, and asserts the groups, the carried count, the wait and wall sums, the launch time and share, and the absent launch time for a group without lists, plus rows where a `slug` keeps only that slug's groups and a `since` drops a record started before it.
It also pins the `--since` refusal through `RunCLI` with `--since 24-10-2026`, and `ReadRunRecords` over a missing directory returning no records and no error.
The gatecli doc gains a Report section, the gateslot doc's Run log section names the reader, and `docs/overview.md` lists the verb.

**ImpactSummary:** Adds one read-only verb and one record reader; no existing verb or record changes.
