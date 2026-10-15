# tokencount reads usage and cost through the shuttle engines

`tools/tokencount` no longer parses Claude Code's transcripts itself.
It locates and reads every session, fork and orch transcript through the engine that recorded it, and prices each message through that engine.
tokencount now holds no transcript path layout, no transcript line format, no price table and no instruction-file names.
A later `codexengine` makes its sessions countable by implementing the same optional capabilities, with no change to tokencount.

For an all-Claude run set the report is unchanged: tables, columns, figures, weight formula, attribution rules and calibration output stay as they are, bar the three wording changes below.

## What changes

- `shuttleengine` declares the neutral read capabilities:
  - a locator that lists the sessions an engine recorded under a worktree path, with each session's forks and last-modified time;
  - a reader that yields a session's entries as neutral records, carrying a per-message usage record (input, cache write, one-hour part of cache write, cache read, output);
  - a pricing capability that prices one message from its model and usage, or reports it unpriced.
- `claudeengine` implements the locator, the reader and pricing.
  The Claude price table, its source citation and its long-prompt rate card move there from `tools/tokencount/cost.go`.
  `SessionUsage` and the new reader share one parse of Claude's usage lines, ending the duplicate that `tools/tokencount/count.go` carried.
- `internal/shuttleengines` gains `RecordReaders`, which returns each registered engine's read capabilities (locator, reader, pricing, instruction files), built without a shuttle config.
- `tools/tokencount` takes that read set through a seam in its entry point and deletes its Claude parser, its `~/.claude/projects` resolution and its `-projects` flag.
- Calibration reads the instruction files named by the Webster session's engine, at the run's base commit, instead of `CLAUDE.md` by name.
- Docs: tokencount's package comment and the `doc.go` of `shuttleengine`, `claudeengine` and `shuttleengines`; `PATTERN-shuttle-provider-seam` gets a line for the read set.

Observable report changes:

- A header line names a registered engine that lacks the locator, since its sessions are not counted.
  It prints only when such an engine exists.
- The `-no-cutoff` header line and the flag's help text say every located session of each slug counts, in place of a Claude project-directory term.
- A no-runs or no-sessions error names the engines asked in place of a projects directory.

## Design decisions

- **Sessions are found per worktree path, by every registered engine.**
  A run whose roles ran on different engines is the union of each engine's sessions, and each session is read through the engine that located it.
- **The reader yields neutral entries; tokencount keeps every lyx-format rule.**
  Provider rules stay in the engine: streamed-message last-line-wins, sidechains, interrupt and local-command markers, compaction summaries, custom titles, the launch-line wording, the fork layout.
  Role from strand name, card pointer lines, the begin-batch envelope, orch run attribution, the time cutoff and cross-transcript dedupe (keyed by engine name and message key) stay in tokencount.
- **An engine without pricing leaves messages with nonzero usage unpriced, never free.**
- **The offline read set hands out adapters, not engines.**
  Each capability value is a small adapter in `shuttleengines` forwarding to an unexported engine, so no type assertion recovers an `Engine` that can start, open, send to or stop a run.
- **Failures stay loud.**
  An engine with no root or no session under a worktree contributes nothing.
  A locator error other than absence, an unreadable located session, or a session that vanished after locating fails the count with an error naming it.
  Malformed or half-written lines are skipped by the engine reader.
- **The default run selection no longer reads a projects directory.**
  Candidates are the slugs with an archive tag in the weft repository, located through every engine, and ranked by the newest last-modified time among their located sessions.
  A candidate with no located session is skipped, so a slug whose transcripts the provider expired does not fail the default report.
- **The repeated-lines figure keeps its meaning.**
  It sums each session's collapsed repeat count from the reader and tokencount's cross-transcript repeats.
- Tests: tokencount tests run parallel over a test-local fake read set and never write a provider's transcript format; Claude-format cases live in `claudeengine`'s tests.
  `cmd/lyx/testdata/test-budget.yaml` is raised for the packages the new tests grow.

## To check by hand

- `engine-model-choice` runs in parallel and edits `websterengine/merriambase.go` and `round-N-usage.yaml`, which `SessionUsage` feeds.
  Whichever lands second re-checks that calibration and `MerriamBase` list the same instruction files, and that round usage readings are unchanged.
- The neutral entry is shaped against Claude's transcripts alone.
  Codex's `token_count` events may not map onto per-message keys and final-record usage; `codex-engine` must implement the locator, reader and pricing for its sessions to count at all.
- The weight column's coefficients (1.25x writes, 0.1x reads, 5x output) remain Claude-shaped; a Codex run's weight will rank on the same scale.
- Run `lyx tokencount` against a real finished run set and compare with a report from before this change; the tests use fixtures and a fake read set only.
