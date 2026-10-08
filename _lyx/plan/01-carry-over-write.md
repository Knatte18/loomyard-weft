# Card 1 — carry-over-write

**Create:**
- `internal/discussionparser/carryover.go`
- `internal/discussionparser/carryover_test.go`

**Edit:**
- `internal/discussionparser#AppendDecision`
- `internal/discussionparser/doc.go`

**Intent:**
Add the decision record's second write path (D2): `WriteCarryOver(decisionRecordPath string, entry CarryOver) error` in a new `carryover.go`, beside `AppendDecision`, so `PATTERN-sole-parsers` holds.
It writes one review segment's carry-over entry into the record's `## Open risks` section, replacing it, appending it, or removing it.

The data shape is owned here, and the caller passes data, never rendered text:

- `CarryOver` carries `Segment` (the segment name, for example `Discussion-Review`), `Round int`, `Closing CarryOverClosing`, `ReviewPath` and `FixerReportPath` (anchor-relative, slash-separated), and `Findings []CarryOverFinding`.
- `CarryOverClosing` has two values: `CarryOverConverged` (a `CONVERGED` verdict) and `CarryOverAccepted` (a recorded circling accept).
- `CarryOverFinding` carries `Key`, `Class`, `Severity` (strings; the caller passes `unlabelled` for a missing label) and `Rounds []int`, the ledger rounds the key was open in.
- `ErrNoOpenRisksHeading` is the sentinel for a record with no `## Open risks` heading.

Rendering, deterministic so a replay writes the same bytes (no date, no clock):

- An opening line `<!-- lyx:carry-over <segment> -->`, the entry's bullets, and a closing line `<!-- /lyx:carry-over <segment> -->`.
- A first bullet naming the segment, the round and how it closed: under `CarryOverConverged` each listed finding was addressed by the round's fixer and no fresh reviewer has seen the fix; under `CarryOverAccepted` a listed finding may also be unfixed.
- A line naming the review and fixer-report paths.
- One nested bullet per finding, in the order given: key, class, severity and the rounds it was open in.
- Every string field (key, class, severity, paths) passes through one sanitizer: line breaks and other control characters become spaces and backticks are escaped as `` \` ``, so a field renders on one line and can never open a heading, a fence or a marker line.
  Keys are rendered as plain text, never inside a code span.
- A `Segment` that is empty or holds anything but letters, digits and hyphens is an error, since it is spelled inside the marker lines.

Splice rules:

- A marker line is a line equal to one of the two marker texts after trailing whitespace is trimmed, counted only inside `## Open risks` (from its heading to the next H2) and outside fenced blocks, using `decisionsInsertOffset`'s fence rule.
  Marker text inline in a sentence, in another section or in a fence is ordinary prose and stays byte-identical.
- With a segment entry present, the bytes from its opening line through its closing line are replaced; with none, a new entry is appended after the section's last non-blank line, preceded by one blank line; with no findings, the segment's entry and the blank line before it are removed, so an append followed by a removal restores the original bytes.
  A record that holds no entry for the segment and gets no findings is left byte-identical and not rewritten.
- A malformed marker set in `## Open risks` is refused before anything is written: an opening line with no closing one, a closing line with no opening one, two pairs for one segment, a closing line naming another segment than the open one, or a pair opened inside another.
- Before replacing the file, the new bytes are re-read and checked: the same H2 heading sequence as the prior file, at most one well-formed pair per segment and exactly one for the written segment (none after a removal), and every line outside the replaced span equal to the corresponding line of the prior file.
  The replaced span is the old entry's lines and the new entry's lines on a replace, the appended blank line and entry on an append, and the removed blank line and entry on a removal; so the check covers ordinary prose and other segments' entries alike, and a splice that damaged any other line fails it.
- The new bytes go to a sibling temporary file renamed over the record, so a failed write or check leaves the record unchanged.
- Every refusal and I/O error is prefixed with the record's path, so the caller's error names the file; a missing record wraps `os.ErrNotExist`.

Update `AppendDecision`'s doc and the header of `append.go`, the file that declares it, which call it the only write path into the record, and the package doc in `doc.go`, to name both write paths; no signature changes.

`carryover_test.go` covers, table-driven where the rows share a shape: append into an empty and a populated section, replace, remove, each asserting every line outside the entry equals the prior file's, a second segment's entry left byte-identical, a replay writing identical bytes, marker text inline in a sentence, in `## Decisions` and in a fence settling normally with that text unchanged, each malformed marker set refused with the path in the error and the file unchanged, no `## Open risks` heading refused with `ErrNoOpenRisksHeading` and the file unchanged, a key holding a line break, a backtick and a marker string rendered on one line without ending the entry early, and the written record passing `Validate`.

**ImpactSummary:** Adds a new exported write path to the stdlib-only discussionparser leaf; AppendDecision's behavior is unchanged and only its doc comment moves.
