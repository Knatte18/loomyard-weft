# Card 19 — pause-intent-entries

**Create:**
- `plan:internal/tendercli#PauseIntentEntries` -> `func PauseIntentEntries(location *lymcwd.Location, slugs []string) error`
- `plan:internal/tendercli#RecordPauseIntent` -> `func RecordPauseIntent(location *lymcwd.Location, slug string) error`
- `plan:internal/tendercli#TestPauseIntentEntries` -> `func TestPauseIntentEntries(t *testing.T)`

**Edit:**
- `internal/tendercli#commitRunIntent` -> `func commitRunIntent(prime *lymcwd.Location, slug, message string, stderr io.Writer, returnOwed bool, write func() (found bool, err error)) error`
- `internal/tendercli#hubRunIntents`
- `internal/tendercli/doc.go`

**Uses:**
- `internal/tendercli#owedIntentPush`
- `internal/pairteardown#ReadBattenRecord`

**Intent:** Give the catch-up and the upgrade's first read the two pause-intent steps they wire (D4 step 5, D5's upgrade).

`PauseIntentEntries(location, slugs)` writes this machine's pause record entries from the pause intents of the runs it owns:
for each slug whose batten record on the prime holds a pause intent and whose scope has no live entry, it pauses the pair through `tenderengine.Pause` with the default caller and the pause grace, so its local processes hold as the intent's run does.
A slug without an intent writes nothing; a failed read or pause of one slug is joined into the error and the others go on.

`RecordPauseIntent(location, slug)` records a pause intent on the run's batten record and pushes it synchronously, as `lym tender pause` records one through `hubRunIntents`' `Record` seam, with `By` the strand or `EXTERNAL`.
Unlike the verb, an owed push is an error: `commitRunIntent` gains a flag saying whether an owed push is answered on stderr (the verbs) or returned (this caller), so the upgrade never drops an entry whose intent is not on origin.
A push that hands the run over returns `PushRun`'s error wrapping `fabricengine.ErrHandedOver` unchanged, so the upgrade reads the run as owned elsewhere.

`TestPauseIntentEntries`, tier1 over a hub fixture: a run with an intent gets an entry, one without gets none, one with a live entry keeps it alone, and a run whose batten record does not read is joined into the error while the next run still gets its entry; `RecordPauseIntent` with origin unreachable returns the owed push's error.
`doc.go` names both functions and their callers.

**ImpactSummary:** Two new exported functions; commitRunIntent gains a flag its two existing callers pass as before.
