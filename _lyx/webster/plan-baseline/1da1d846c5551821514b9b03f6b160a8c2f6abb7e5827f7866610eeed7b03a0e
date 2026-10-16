# Card 1 — journal-tail-repair

**Edit:**
- `internal/creelengine#repairTail`
- `internal/creelengine#foldJournal`
- `internal/creelengine#TestJournal_Fold`
- `internal/creelengine/doc.go`
- `tools/sandbox/SANDBOX-CREEL-SUITE.md`

**Create:**
- `plan:internal/creelengine#lastNonBlankLine` -> `func lastNonBlankLine(lines [][]byte) int`
- `plan:internal/creelengine#parseRecord` -> `func parseRecord(line []byte) (record, error)`

**Uses:**
- `internal/creelengine#openJournal`
- `internal/creelengine#recordLine`

**Intent:** Fix #677 (D1): the tail repair at journal open cuts a torn last line whether or not it ends in a newline.

`lastNonBlankLine` returns the index of the last line of `bytes.Split(data, "\n")` whose trimmed text is not empty, or -1 when there is none.
`parseRecord` is the one parse test: `json.Unmarshal` into a `record`, so a line that is valid JSON but no record, such as `[1]`, fails it.
`foldJournal` uses both for its existing skip of the newest file's torn last line, in place of its inline loop and its inline `json.Unmarshal`, so the line the fold skips is by construction the line the repair cuts.

`repairTail` reads the newest day file, splits it into lines and takes `lastNonBlankLine` as the tail.
With no such line, or a tail that passes `parseRecord` and ends in a newline, it changes nothing.
A tail that passes `parseRecord` but is the file's last bytes with no newline gets the newline appended, as today.
A tail that fails `parseRecord` is cut: the file is truncated to the end of the line before it, newline included, which also drops any blank lines after the tail, and to zero bytes when the tail is the file's only non-blank line; the existing Warn names the file and the byte count cut.
A malformed line anywhere else is never touched by the repair, and `foldJournal`, which `OpenStore` runs before `openJournal`, still fails the start naming the file and line with its way forward.
Bound: only the newest file's last non-blank line can be cut, the line the fold already skips, so no record a fold applies is removed.

Test: `TestJournal_Fold` gains an optional per-row check of the repair: after the fold, the row opens the journal with `openJournal` over the folded pool on a `testClock` at `journalEpoch`, compares the newest day file's bytes with the row's wanted content, appends one record, and folds the directory again with no error and that record applied.
The fold's warning count stays as it is, and the repair's own Warn is asserted to carry the byte count.
Rows:
- the existing torn last line with no newline is cut, and the next append follows the last valid line;
- a torn last line that ends in a newline is skipped by the fold and cut, and the refold after the append has no error (the #677 case, which fails today with the torn line become a middle line);
- a torn last line followed by blank lines is cut with the blank lines;
- a newest file whose only line is torn is cut to empty, and the append writes the high-water record first;
- a last line `[1]`, valid JSON but no record, is skipped and cut;
- a complete last line missing its newline gets the newline and keeps its record;
- a complete last line followed by blank lines is left byte for byte as it was;
- a newest file holding only blank lines is left byte for byte as it was, and the append after it writes no high-water record into a file that already has content.
The existing rows for a malformed middle line and a torn last line of an older file keep pinning that those still fail the load.

Docs in the same commit: the Storage paragraph of `internal/creelengine/doc.go` says a torn last line of the newest file, with or without its newline, is skipped by the fold and cut when the journal opens; the C10 watch line of `tools/sandbox/SANDBOX-CREEL-SUITE.md` says a torn tail is cut whether or not it ends in a newline, and a restart after the next append starts cleanly.

**ImpactSummary:** Two private journal functions in `internal/creelengine` share one new last-line helper and parse test; `openJournal` and `OpenStore` keep their signatures, and only `TestJournal_Fold` and two doc texts change beside them.
