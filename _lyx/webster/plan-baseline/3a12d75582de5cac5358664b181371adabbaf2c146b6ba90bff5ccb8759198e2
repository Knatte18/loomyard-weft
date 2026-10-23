# Card 10 — fixer-attempts

**Create:**
- `plan:internal/landingshed#FixerAttempts` -> `type FixerAttempts struct`
- `plan:internal/landingshed#FixerAttemptsPath` -> `func FixerAttemptsPath(scratchDir, row string) string`
- `plan:internal/landingshed#ReadFixerAttempts` -> `func ReadFixerAttempts(path string, entry int) (FixerAttempts, error)`
- `plan:internal/landingshed#WriteFixerAttempts` -> `func WriteFixerAttempts(path string, rec FixerAttempts) error`
- `plan:internal/landingshed#RemoveFixerAttempts` -> `func RemoveFixerAttempts(path string) error`
- `plan:internal/landingshed#TestFixerAttempts` -> `func TestFixerAttempts(t *testing.T)`

**Uses:**
- `internal/landingshed#writeBytesAtomic`

**Intent:** Keep the fixer turns one row entry spent in a record in the run's scratch directory, so no resume grants a fresh cap (D4).
`FixerAttempts` holds `Entry`, the row entry the record belongs to, `Spent`, the fixer turns spent, `Baseline`, the HEAD a cap measures from, `AlignFrom`, the parent tip a Merge-In entry reads the parent's changes from when the merge record holds no aligned tip, and `Halt`, the note of the halt that last stopped the entry (`spent`, `over-cap` or empty).
An entry has no attempts left when `Spent` reaches its row's cap, whatever `Halt` notes, so a turn that ends any way on the last attempt leaves the entry exhausted, and Merge-In and Finalize key their exhausted-entry path on `Spent`, save that a resume of an entry noted `over-cap` takes the over-cap resume first (cards 26 and 29).
`FixerAttemptsPath(scratchDir, row)` names one file per row, `<row>-fixer-attempts.json`, under the told scratch directory (`PATTERN-told-geometry`).
`ReadFixerAttempts(path, entry)` returns the record when it names `entry`, so a resume continues its own entry; a record naming another entry, as a re-entry from another row finds, and an absent file both return a zero record for `entry`, which starts at zero spent attempts.
The record is JSON written through `writeBytesAtomic`, and an unknown key is tolerated (`PATTERN-tolerant-records`).
An undecodable file is an error naming it; Merge-In and Finalize turn that error into a Stuck with Halt whose reason opens with `LandingStopPrefix`, naming the file, so the hand-back mark sends it to the orch and no agent driver lifts it, with the way forward to remove the file by its path, which grants the entry a fresh cap, then `lyx loom resume` (cards 26 and 29 pin that halt).
`WriteFixerAttempts` writes it atomically and `RemoveFixerAttempts` removes it, an absent file being no error.

The test round-trips a record, reads it back under its own entry, reads a zero record under another entry and from an absent file, tolerates an unknown key, refuses an undecodable file and removes the file twice.
