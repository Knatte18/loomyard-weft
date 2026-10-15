---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: _lyx/plan/02-claude-transcript-reader.md:88
    summary: the line-keyed fallback message key is pinned by no row that can observe it, and the SessionUsage row that names it passes under any keying
    fix: >-
      In card 2, define the fallback key's n as the 1-based line number of the record in its transcript file.
      Add a TestReadSession row with a main transcript and a fork transcript that each hold an id-less, uuid-less assistant usage line at the same line number,
      and assert the two entries' MessageKey values are line:<main ID>:<n> and line:<main ID>/<fork ID>:<n>, so they differ.
      In the TestSessionUsage_SumsParentAndForks bullet, drop the keyed line:... clauses from both id-less rows and state that they pin only that each id-less line counts,
      since SessionUsage returns sums and parses each transcript with its own per-message map; the fork-side row beside a parent line of the same number then adds nothing the parent-side row does not, so delete it or keep it as the fork-side counting case only.
---

### [LOW] The `line:` fallback key is unobserved by any named test

**Location:** `_lyx/plan/02-claude-transcript-reader.md:88` (the `TestSessionUsage_SumsParentAndForks` bullet), with `:62` (the key rule) and `:82` (`TestReadSession`).
**Issue:** Round 1 introduced the third fallback, `line:<transcriptKey>:<n>`, so two id-less messages without a uuid never merge, in one transcript or across transcripts.
The key scheme itself is collision-free by construction: `transcriptKey` is the session `ID` for a main transcript and `<session ID>/<fork ID>` for a fork, and the slash cannot occur in a session `ID`, so no two transcripts of one engine share a prefix.
No test can fail if that breaks.
`SessionUsage` returns only sums and `parseTranscript` keeps its per-message map per transcript, so the two id-less `TestSessionUsage_SumsParentAndForks` rows, which the card says are "keyed" by `line:...`, pass under any keying, the fork row beside a parent line of the same number included.
`TestReadSession`'s id-less row carries uuids and asserts keys only by uuid, and today's `TestCountRun` fixture that the golden reproduces holds no id-less line (every `assistant(...)` there carries an id), so the golden pins no `line:` key either.
Card 5's cross-transcript dedupe rows run over hand-built reads and never reach the Claude key.
The card also leaves `<n>` undefined, so the implementer picks the numbering the unwritten test would have to pin.
The decision record's D2 requires the key to be unique per message and tokencount's cross-transcript dedupe to rest on it, so this is the one key rule of the focus area with no guard.

## Focus verdicts

- The golden `tokencount-run-reads.json` lives in `internal/testkit/shuttlefake/`, which both `claudeengine`'s tests (already importers of the kit, so no import cycle) and `tools/tokencount`'s tests read; the transcripts under `claudeengine/testdata/tokencount-run/` are read by `claudeengine` alone.
  No fixture used by two or more packages sits outside the kit, so `PATTERN-testkit` holds; not a defect.
- Two distinct messages from different transcripts of one run cannot get the same key by the plan's rules: a message id is the provider's, a uuid is per line, and the fallback is prefixed by a transcript key unique per transcript; `messageKey` adds the engine name.
  The only gap is the test coverage F1 names.
- Every `TestSessionUsage_SumsParentAndForks` row matches what `SessionUsage` does today under `subagents/`: a directory entry is skipped and the reading stays known (`entry.IsDir()` in `usage.go`), a `subagents` path that is a file makes `os.ReadDir` fail with an error that is not `IsNotExist`, so the reading is unknown, and each id-less line counts.
  The one flaw is the unobservable keying clause F1 names.
