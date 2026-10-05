# Card 12 — `discussionparser` appends a post-Discussion decision

**Create:**
- `plan:internal/discussionparser#AddedDecision` -> `type AddedDecision struct`
- `plan:internal/discussionparser#AppendDecision` -> `func AppendDecision(decisionRecordPath, supportLogPath string, d AddedDecision) ([]Finding, error)`
- `internal/discussionparser/append.go`
- `internal/discussionparser/append_test.go`

**Edit:**
- `internal/discussionparser/doc.go`

**Intent:**
D8: the decision-record append lives in `internal/discussionparser`, the record's sole parser and now its sole appender (`PATTERN-sole-parsers`), and stays stdlib-only.

`AddedDecision` carries `By`, `Title`, `Decision`, `Rationale` and `Date` (`time.Time`).
`AppendDecision`:
1. Reads the decision record; a missing file is returned as an error wrapping `os.ErrNotExist`, so the caller can refuse with its own way forward.
2. Errors, writing nothing, when `By`, `Title`, `Decision` or `Rationale` is empty after trimming.
3. Inserts, at the end of the `## Decisions` section (immediately before the next H2 line, or at end of file when it is the last section), one entry: a blank line, the H3 `### Added after Discussion (<By>, <Date in UTC as YYYY-MM-DD>): <Title>`, a blank line, `Decision: <Decision>`, a blank line, `Rationale: <Rationale>`.
   Every byte before and after the insertion point is kept unchanged.
   A record with no `## Decisions` heading is an error, and nothing is written.
4. Writes the file, then runs `Validate`.
   On findings it restores the prior bytes and returns the findings with a nil error;
   on an I/O error during the check it restores and returns the error.

Bound: append-only, one entry per call; it never edits or removes existing text and never touches the support log.
`doc.go` names the package as the record's sole appender as well as its reader.

Tests in `append_test.go` over a temp record and support log: the entry lands as the last block of `## Decisions` with the exact heading and lines, and the text before and after is byte-identical; appending to a record whose `## Decisions` is the last section works; an empty field and a missing `## Decisions` heading write nothing; a missing record returns an error satisfying `errors.Is(err, os.ErrNotExist)`; over a record that already lacks a required section, the check's finding is returned and the original bytes are restored.

**ImpactSummary:** Additive to the leaf package; no caller until Card 13.

**Commit:** 12: discussionparser appends a post-Discussion decision
