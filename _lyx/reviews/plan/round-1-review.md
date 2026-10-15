---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/02-claude-transcript-reader.md:29
    summary: The reads.json golden is a fixture shared by claudeengine's and tokencount's tests but sits under claudeengine's testdata, which PATTERN-testkit forbids
    fix: In card 2's Create list, replace `internal/shuttleengine/claudeengine/testdata/tokencount-run/reads.json` with `internal/testkit/shuttlefake/tokencount-run-reads.json` and `plan:internal/testkit/shuttlefake#TokencountRunReads` -> `func TokencountRunReads() []shuttleengine.SessionRead`, and say in its Intent that the kit embeds the JSON and decodes it into the reader output the golden row compares against, while the transcripts stay under claudeengine's testdata. In card 5, add `plan:internal/testkit/shuttlefake#TokencountRunReads` to Uses and change the TestCountRun bullet (line 126) to take its main fixture from `shuttlefake.TokencountRunReads()` instead of claudeengine's testdata path. Checked with validate-plan --overlay; no new blocking finding.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/02-claude-transcript-reader.md:60
    summary: The line:<n> fallback message key is unique only within one transcript, so tokencount's cross-transcript dedupe by engine and message key merges distinct id-less messages that share a line number, against D2's key unique per message
    fix: In card 2's MessageKey bullet, make the last fallback unique across the engine's transcripts by prefixing the transcript, for example `line:<transcript ID>:<n>` with the ID the TranscriptRead carries, and update the TestReadSession and TestSessionUsage_SumsParentAndForks id-less rows (line 83) to expect the new key shape.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/02-claude-transcript-reader.md:83
    summary: The new TestSessionUsage row that expects a directory in a fork transcript's place to make the reading unknown contradicts SessionUsage today, which skips a directory entry under subagents/, so it pins a behavior change D5 rules out
    fix: In card 2's TestSessionUsage_SumsParentAndForks bullet, replace the row "a fork transcript that cannot be read (a directory in its place) makes the reading unknown" with "a directory named like a fork transcript under subagents/ is not a fork, and the reading stays known with that fork left out", and say in the SessionUsage paragraph that a directory entry under subagents/ is skipped, as today.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/05-tokencount-reads-through-engines.md:18
    summary: Card 5 re-signs messageCost and run but lists only cost.go and main.go, so neither re-signature is declared with the arrow and caller-uncovered never runs on them
    fix: No admissible target change found. Adding `tools/tokencount#messageCost` and `tools/tokencount#run` with re-sign arrows beside `tools/tokencount/cost.go` and `tools/tokencount/main.go` is refused by redundant-file-target, and dropping the file targets loses the file comment and package comment edits the Intent prescribes. Leave the targets, and state both new heads in card 5's ImpactSummary next to the existing caller claim, so the re-signatures are visible in the blast-radius line.
---

### [BLOCKING] Shared golden fixture placed under the package it covers

**Location:** `_lyx/plan/02-claude-transcript-reader.md:29`, used again at `_lyx/plan/05-tokencount-reads-through-engines.md:126`

**Issue:** Card 2 commits `reads.json` under `internal/shuttleengine/claudeengine/testdata/tokencount-run/`, and card 5's `TestCountRun` in `tools/tokencount` decodes the same file.
That makes it a fixture shared by two packages.
`PATTERN-testkit` (background `pattern/PATTERN-testkit.md`) says fixtures used by two or more packages go through one kit under `internal/testkit/<kit>/`, never under the package they fake.
Under this round's rules a constraint violation is BLOCKING.
The design idea is sound: one golden feeds both halves of the fixture-reproduction criterion, so they cannot drift.
Only where the file lives breaks the rule.
`internal/testkit/shuttlefake` already imports only `shuttleengine`, so it can hold an embedded golden for both test packages without an import cycle.
The proposed targets passed `lyx loom validate-plan --overlay` with no blocking finding beyond the `{}` baseline.

### [MEDIUM] The id-less key fallback collides across transcripts

**Location:** `_lyx/plan/02-claude-transcript-reader.md:60`

**Issue:** D2 says the engine makes the key of a message without an id "unique per message".
Card 5 dedupes across every transcript of a run by engine name plus `MessageKey`.
For a line with neither an id nor a uuid, card 2's fallback is `line:<n>`.
Two such messages on the same line number of two transcripts (for example two sessions, or a session and its fork) would get the same key, and tokencount would count only one of them and add one to `Duplicates`.
Today's tokencount keys by uuid, and `SessionUsage` counts each id-less line on its own.
The new key fixes the merge inside one file and brings it back across files.

### [MEDIUM] A SessionUsage row pins a behavior change D5 rules out

**Location:** `_lyx/plan/02-claude-transcript-reader.md:83`

**Issue:** D5 keeps `SessionUsage`'s observable behavior.
Today `SessionUsage` skips a directory entry when it lists `subagents/` (`internal/shuttleengine/claudeengine/usage.go:293`, `entry.IsDir() || !strings.HasSuffix(...)` then `continue`).
A directory in a fork transcript's place therefore leaves the reading known with that fork left out.
The planned row expects the reading to be unknown.
An implementer either changes the behavior to make the row pass, against D5, or ends up with a row that fails.
The other two new rows (id-less lines without uuids, and a `subagents` path that is a file) match today's code.

### [LOW] Two re-signatures carried only by file targets

**Location:** `_lyx/plan/05-tokencount-reads-through-engines.md:18`

**Issue:** The Intent changes `messageCost` to `func messageCost(pricer shuttleengine.MessagePricer, model string, u shuttleengine.MessageUsage) Cost`, and `run` to `run(args []string, stdout io.Writer, reads ReadSet) error`.
Neither has a member glyph, so neither carries the re-sign arrow, and `caller-uncovered` does not check them.
Every caller I found is in this card: `countFile` and `readTranscript` for `messageCost`, and `main` plus `TestRunRefuses` and `TestRunReadsArchivedShedStatus` for `run`.
So nothing breaks, but the plan's record of the change is incomplete.
The rubric's "member glyphs on an Edit card" check does not fire, because the card also lists member glyphs.
The re-sign-arrow check does not fire either, because there is no glyph to carry an arrow.
`redundant-file-target` blocks the natural fix.

## Focus verdicts

- Card 1 granularity: not a defect.
  Every neutral type and constant is on the one card that declares the seam.
  The granularity rule decides how symbols are split across cards, not how many glyphs a card lists.
- Card 2 test economy: not a defect.
  `TestReadSession` pins the reader's entries, flags, keys and repeat counts, which no current test reads.
  `TestSessionUsage_SumsParentAndForks` covers only the summed reading.
  `TestLocateSessions` pins listing, sorting and the absence rules.
  `TestClaudeProjectDirFor_EncodesNonAlnumBytes` covers only the encoding.
  The fixtures feed the reproduction row, not a separate test.
- Cards 4 and 5 test economy: not a defect.
  `readingEngine`, `fakeTranscripts`, `fakePricer`, `oneEngine` and the entry builders are fixtures reached through `TestRecordReaders` and the existing tokencount tests.
  Card 5 adds no top-level test.
  Card 4 adds one and raises the `internal/shuttleengines` budget row from 5.
  Card 2 and card 3 bring `claudeengine` from 110 tests to its row of 113.
- Card 5 Edit glyphs: every member glyph whose signature changes carries the arrow.
  `main.go` and `cost.go` sit beside member glyphs, so the "only file paths" check does not fire.
  The undeclared `messageCost` and `run` re-signatures are F4.
- Card order: not a defect.
  Card 6 edits `CountRun`, `RunTally`, `main.go` and `TestCountRun` after card 5, and uses card 5's `ReadSet` and `oneEngine`.
  No card uses a symbol a later card creates.
- Card 4 attack surface: not a defect.
  The Intent says no type assertion recovers an `Engine` and the set exposes reading alone, and `TestRecordReaders` asserts it.
  The `PATTERN-shuttle-provider-seam.md` bullet agrees.
- Fidelity: D1 to D10 and the Constraints are each carried by a card.
  The D7 header line, the `-no-cutoff` help and header text, both error wordings and the `-projects` deletion are in card 5.
  D8 is in card 6.
  The departures from the decision record are F2 and F3.
  No card adds scope the record does not license.
- Verify: `go test ./...` and `go test -tags integration ./...` run every targeted package, `cmd/lyx` and `internal/testkit` included.
  The order matches the spec's verify model.
