# Plan round 1 — fixer report

## Changes

- **F1** (BLOCKING), fixed as proposed, plus one addition.
  Card 2's Create list swaps `claudeengine/testdata/tokencount-run/reads.json` for `internal/testkit/shuttlefake/tokencount-run-reads.json` and `plan:internal/testkit/shuttlefake#TokencountRunReads` -> `func TokencountRunReads() []shuttleengine.SessionRead`.
  The fixture-reproduction bullet says the kit embeds the JSON and decodes it into the reader output the golden row compares against, while the transcripts stay under claudeengine's testdata.
  Card 5 lists the kit function under Uses, and its `TestCountRun` bullet takes the main fixture from `shuttlefake.TokencountRunReads()`.
  Addition: card 2 also edits `internal/testkit/shuttlefake/shuttlefake.go`, which declares the function and whose package comment ("fakes the two seams") would otherwise no longer describe the kit.
  `go list -deps ./internal/testkit/shuttlefake` shows no `claudeengine` or `tokencount` dependency, so neither test package hits an import cycle.
- **F2** (MEDIUM), fixed differently.
  The proposed key `line:<transcript ID>:<n>` uses the `TranscriptRead` ID, which for a fork is its file name.
  That name is unique only within its session's `subagents/`, so two sessions' `agent-1.jsonl` forks could still collide.
  `parseTranscript` gains a `transcriptKey` parameter: the session ID for a main transcript, `<session ID>/<fork ID>` for a fork.
  The fallback key becomes `line:<transcriptKey>:<n>`, and both `ReadSession` and `SessionUsage` pass it.
  The glyph signature and the Intent's signature are updated.
  `TestSessionUsage_SumsParentAndForks`'s id-less row expects `line:<session ID>:<n>`, and a new row there covers the cross-transcript case: an id-less fork line on the same line number as a parent line counts beside it.
  `TestReadSession`'s id-less row is keyed by uuids, so it does not reach the fallback and stays unchanged.
- **F3** (MEDIUM), fixed as proposed.
  The row now expects that a directory named like a fork transcript under `subagents/` is not a fork, and the reading stays known with that fork left out.
  The `SessionUsage` paragraph says a directory entry under `subagents/` is skipped, as today, which matches `internal/shuttleengine/claudeengine/usage.go:293`.
  "gains three rows" became "gains rows", since the row count changed.
- **F4** (LOW), fixed as proposed.
  Card 5's ImpactSummary states both new heads.
  It names their callers as checked with grep: `countFile`, `readTranscript` and `TestEstimatedCost` for `messageCost`; `main`, `TestRunRefuses`, `TestRunReadsArchivedShedStatus` and `TestCalibrateGitBackedMatchesInMemory` for `run`.
  The review's list left out the two test callers.

`lyx loom validate-plan` reports `ok: true` with informational findings only.

## Disputed

None.

## Deferred

None.
