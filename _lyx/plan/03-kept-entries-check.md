# Card 3 — kept-entries-check

**Create:**
- `plan:internal/discussionparser#CheckAddedDecisionsKept` -> `func CheckAddedDecisionsKept(record []byte, versions [][]byte) []Finding`
- `plan:internal/discussionparser#checkAddedDecisionDropped` -> `const checkAddedDecisionDropped = "added-decision-dropped"`
- `plan:internal/discussionparser#TestCheckAddedDecisionsKept` -> `func TestCheckAddedDecisionsKept(t *testing.T)`

**Edit:**
- `internal/discussionparser/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/discussionparser#ParseAddedDecisions`
- `internal/discussionparser#Finding`

**Intent:** Add the parser half of D4's history check: a record that dropped an `Added after Discussion` entry any earlier version of it held is reported entry by entry, so the writer can restore it.

`CheckAddedDecisionsKept` lives in `internal/discussionparser/append.go` beside `ParseAddedDecisions`.
It parses `record` and each of `versions` with `ParseAddedDecisions` and returns one `Finding` with `Check` set to `checkAddedDecisionDropped` for each distinct entry, by author, date and title, that some version holds and `record` does not, in the order of first appearance across `versions`.
Each finding's `Detail` names the full heading as `### Added after Discussion (<by>, <date>): <title>` and says to restore the entry from the record's history, since an added entry is never the writer's to remove.
A `record` whose added-decision heading does not parse is one finding with the parse error as its detail, never an error, because the writer can fix it.
A version whose heading does not parse contributes no entries: only `lyx loom decision add` writes these headings, and a garbled historical copy must not block the run.
No versions, or versions holding nothing the record lacks, return no findings.
It reads nothing from disk, so it returns no error.

`TestCheckAddedDecisionsKept` is one table: no versions; a version whose entry the record keeps; two entries in history and the record dropping one, which yields exactly one finding naming that heading; an entry with the same title but another date counted as distinct; an unparseable heading in the record, which yields a finding; and an unparseable heading in a version, which yields none.
The package doc names the check beside `AppendDecision` and `WriteCarryOver`.
The new top-level test raises `internal/discussionparser`'s row in `cmd/lyx/testdata/test-budget.yaml` from 18 to 19.

**ImpactSummary:** one new pure function in the stdlib-only discussionparser leaf and its budgeted test; no caller yet.

**Commit:** `3: discussionparser reports added entries a record dropped`
