# Decision record — engine-usage-readers

## Goal

`tools/tokencount` reads token use, time and cost through the shuttle engine that recorded each session, not by parsing Claude Code's transcripts itself.
After this task tokencount holds no provider specifics: no transcript path layout, no transcript line format, no price table and no instruction-file names.
A later `codexengine` makes its sessions countable by implementing the same optional capabilities, with no change to tokencount.

## Scope

In:

- A neutral transcript-reading capability in `shuttleengine`: a locator that lists the sessions an engine recorded under a worktree path, with each session's forks and last-modified time, and a reader that yields a session's entries as neutral records.
- A neutral per-message usage record in `shuttleengine`, carried on those entries.
- A pricing capability in `shuttleengine`; the Claude price table moves from `tools/tokencount/cost.go` into `claudeengine`.
- `claudeengine` implements the locator, the reader and pricing, and its existing `SessionUsage` shares the one Claude transcript usage parse with the new reader.
- `internal/shuttleengines` hands out every registered engine's read capabilities for offline record reading, built without a shuttle config.
- tokencount reads every session, fork and orch transcript through those capabilities and deletes its own Claude parser, its `~/.claude/projects` resolution and its `-projects` flag.
- Calibration's Merriam texts read the instruction files named by the Webster session's engine instead of `CLAUDE.md` by name.
- One addition beyond the board entry: a report header line naming a registered engine that lacks the locator (D7).
- Docs in the same commit: tokencount's package comment, and the `doc.go` of `shuttleengine`, `claudeengine` and `shuttleengines`.

Out:

- Token totals in `lyx report timing`: the board entry's Out names it as later work, and no board entry for it exists yet.
- `websterengine.MerriamBase`'s live instruction-file read, which `engine-model-choice` owns.
- A `codexengine`, Codex prices or Codex transcript parsing (`codex-engine`).
- Changing what the report measures: for an all-Claude run set its tables, columns, figures, weight formula, attribution rules and calibration output stay as they are, bar three wording changes.
  The D7 header line prints only when an engine is unreadable.
  The `-no-cutoff` header line, and the flag's help text, say every located session of each slug counts, in place of every transcript in each slug's project directory, a Claude layout term.
  The default run selection's no-runs error names the engines asked in place of the projects directory (D10).
  A named run's no-sessions error names the engines asked in place of the project directory (D7).

## Decisions

### D1 — Sessions are found per worktree path, by every registered engine

Decision: The locator capability takes a worktree path and returns the sessions that engine recorded there, each with its fork sessions and the last-modified time of its own record, its forks' left out.
tokencount asks every registered engine that implements it, for each counted worktree (each run's `<hub>/<slug>`, and the prime for the orch), and reads each session through the engine that located it.
"Each run's engine" is realised per session: a run whose roles ran on different engines is the union of each engine's sessions.

Rationale: The worktree path is the one key that survives a finished run's teardown, and it is the key Claude's own layout uses.
Per-session reading follows from `engine-model-choice`, which lets roles of one run resolve to different engines.

### D2 — The reader yields neutral entries; tokencount keeps every lyx-format rule

Decision: The reader yields, per session, its name (the strand name lyx set on it), its repeat count, and its entries in recorded order.
The repeat count is the streamed records it collapsed into their message's last record plus, in a fork, every line of a replayed parent message it skipped.
An entry carries its timestamp, whether it is a user or an assistant entry, whether it starts or ends a turn, a message key and the model, its usage when it is the message's final record, its user texts, its tool calls classed as shell, file read or other with the command or path, its tool results with the call they answer and their text, and the prompt path when the entry is the engine's own launch line.
The message key is the provider's message id, or, for a message without one, a key the engine makes unique per message (Claude's line uuid, as tokencount keys it today).
Every provider rule that decides those fields stays in the engine: Claude's streamed-message last-line-wins rule, sidechain handling, interrupt and local-command markers, compaction summaries, custom-title lines, the launch-line wording, the fork transcript layout.
Every lyx-format rule stays in tokencount: role from strand name, card pointer lines, the begin-batch envelope, the orch's run attribution, the time-bound cutoff, and cross-transcript dedupe, keyed by engine name and message key.

Rationale: tokencount reads far more than usage (turn time, card attribution, the begin-batch base, the Merriam start), and the board entry requires its parser deleted, so the neutral record carries every fact those reads use.
Splitting by who owns the format keeps the seam clean: a provider changes its transcript shape in its engine only, and lyx changes its own prompt formats in tokencount only.

### D3 — One usage record; cache writes split by lifetime

Decision: The neutral per-message usage record holds input, cache-write, the one-hour part of cache-write, cache-read and output tokens.
An engine whose provider reports no cache writes or no lifetime split leaves those at zero.

Rationale: These are the fields tokencount's cost and weight use; nothing else is read.

### D4 — Prices live in each engine, behind a pricing capability

Decision: The pricing capability prices one message from its model and neutral usage, returning a USD figure or "unpriced".
Claude's list-price table, its source citation, its long-prompt rate card and its lifetime-split cache-write pricing move into `claudeengine`.
tokencount keeps its cost tally (sum, per-model unpriced counts, share, rendering) and prices each message through the engine that read its session.
An engine without the capability leaves every message with nonzero usage unpriced, never free.

Rationale: Rate cards are provider facts; the tally and its rendering are report logic.
The existing "unpriced, never free" rule extends to a whole engine unchanged.

### D5 — One Claude transcript usage parse in `claudeengine`, beside `UsageReader`

Decision: The new capabilities sit beside the existing `UsageReader`, which stays as shuttle's per-run reading.
`claudeengine`'s `SessionUsage` and the new reader share one parse of Claude's usage lines, so the duplicate the board entry names disappears rather than moving.
`SessionUsage` keeps its observable behaviour: fork totals, skipping the parent's replayed message in a fork, counting each id-less usage line, an unreadable transcript making the reading unknown.

Rationale: The board entry's "Seen" names the two copies as a duplication to end, and `UsageReader` serves a different caller (a run's finalize) with different failure rules.

### D6 — An offline read set from the registry

Decision: `internal/shuttleengines` gains a constructor returning, per registered engine name, that engine's read capabilities only (locator and reader, pricing, instruction files), built with a zero shuttle config as `SessionNamers` builds engines.
Each capability value in the set is a small adapter in `shuttleengines` that forwards to the engine it holds in an unexported field, so its dynamic type is the adapter and no type assertion on it recovers an `Engine` that can start, open, send to or stop a run.
The adapters are one per capability, not one per engine.
tokencount takes that set through a seam in its entry point, and its tests inject a test-local fake with neutral entries and never write a provider's transcript format.
Claude's locator resolves its projects root as `claudeengine` already does (from `HOME`), and the parsing and locating internals take the root as a parameter, so Claude-format tests run parallel on a temporary root and one test covers the `HOME` default.
The `-projects` flag is deleted; its only users are tokencount's own integration tests, which move to the injected fake.

Rationale: `PATTERN-shuttle-provider-seam` lets only `shuttleengines` construct a concrete engine, and a tool that reads records needs neither `shuttle.yaml` nor a default engine.

### D7 — Failures stay loud in tokencount; absence is not a failure

Decision: An engine whose root does not exist, or that recorded no session under a worktree, contributes nothing.
A named run with no session from any engine fails with the existing "no sessions" error, naming the engines asked; the default selection skips such a run instead (D10).
A locator error other than absence, and a located session whose transcript cannot be read, fail the count with an error naming it.
A located session whose transcript vanished before it was read is unreadable and fails the count the same way.
Its one cause is the provider expiring old transcripts while the count runs; a rerun no longer locates the session and counts without it, so no located session is ever left out of a count that succeeds.
Malformed or half-written entries (a live session's last line) are skipped by the engine reader, as the current parser skips them.
When a registered engine lacks the locator capability, the report header names it on one line, since its sessions, if any, are not counted.

Rationale: tokencount is an operator report, and a silent undercount is worse than a refusal; absence is the normal state of an engine that ran nothing in a worktree.

### D8 — Calibration reads the engine's instruction files at the base commit

Decision: The Merriam texts calibration reconstructs take their instruction files from the Webster session's engine through the existing instruction-file capability, each read at the run's base commit, an absent file adding nothing.
An engine without the capability contributes no instruction texts.

Rationale: This is the board entry's "Calibration reads the engine's instruction files", and it mirrors the change `engine-model-choice` makes to the live `MerriamBase`, so the reconstruction and the live count keep agreeing; `CLAUDE.local.md` is untracked and stays absent at the base commit, as the current assumption says.

### D9 — The repeated-lines figure keeps its meaning

Decision: The "All runs" total line's count of repeated transcript lines skipped sums each session's collapsed repeat count from the reader (D2) and tokencount's cross-transcript repeats, so it counts what it counts today.

Rationale: The figure exists in the report today, and D2 moves half of what feeds it into the engine.

### D10 — The default selection ranks finished runs by located-session recency

Decision: With no slugs named, the candidates are the slugs with an archive tag in the weft repository, the prime left out, each located at `<hub>/<slug>` through every engine's locator.
A candidate's recency is the newest last-modified time among its located sessions (D1), so ranking reads no transcript; for Claude that time is the main transcript file's modification time, the key `recentRuns` uses today.
A candidate with no located session is skipped, never the D7 "no sessions" error, so a finished slug whose transcripts the provider has expired does not fail the default report; a locator error still fails it.
The -last most recent full loom runs are counted, and the skipped not-full-loom runs are named as today.
When no full loom run is found, the error names the hub and the engines asked, not a projects directory.

Rationale: Today's candidates are the hub's project directories under the Claude projects root, which leaves tokencount with the root (Scope).
The archive tags already gate today's candidates, so drawing candidates from them and ranking by the same file time selects the runs today's code selects.

## Constraints

- `PATTERN-shuttle-provider-seam`: every Claude transcript, path and price fact lives in `internal/shuttleengine/claudeengine`; of the engine packages, tokencount imports only `shuttleengine` and `shuttleengines`.
  Its import scan already covers every production file, `tools/` included.
- `PATTERN-told-geometry`: the engine reader is handed the worktree path and derives no worktree geometry; the provider's own home root is the engine's existing resolution; `shuttleengines` stays bound by the entry.
- `PATTERN-test-speed`, `PATTERN-test-economy`: tokencount tests run parallel over a test-local fake; Claude-format cases move to `claudeengine`'s tests rather than being duplicated.
- `PATTERN-comment-line-breaks` applies to the rewritten package comment and every new doc comment.
- Test budget: a card whose new top-level tests push `internal/shuttleengine/claudeengine`, `internal/shuttleengines` or `tools/tokencount` past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.

## Auto-mode assumptions

- Calibration stays Claude-shaped in substance (it measures Webster forks, a Claude-only role per `engine-model-choice`); only its file names and transcript reads go through the engine.

## Open risks

- `engine-model-choice` runs in parallel and edits `websterengine/merriambase.go` and `round-N-usage.yaml`, which `SessionUsage` feeds; whichever lands second re-checks that calibration and `MerriamBase` list the same instruction files and that round usage readings are unchanged.
- The neutral entry is shaped against Claude's transcripts alone; Codex's `token_count` events may not map onto per-message keys and final-record usage, and `codex-engine` must implement the locator, reader and pricing for its sessions to count at all.
- The weight column's coefficients (1.25x writes, 0.1x reads, 5x output) are Claude-shaped; they stay a documented relative ranking, and a Codex run's weight will rank on the same scale.

## Acceptance criteria

- `tools/tokencount` contains no Claude transcript field name, no `.claude` path and no price figure; `cost.go`'s table lives in `claudeengine`; the provider-seam import scan passes.
- For a fixture set of Claude transcripts, the Claude reader yields entries from which tokencount reproduces today's role tokens, cost, working time, repeated-lines figure, fork cards, fork start and peak context, begin-batch base and Merriam start.
- Claude reader scenarios: a message streamed over several lines yields its usage once, from its last line, and counts its earlier lines as repeats; two id-less messages both count; a fork transcript's replayed parent message is skipped in the fork and counts each of its lines in the fork's repeat count; the latest custom title names the session; interrupt markers, local-command output and compaction summaries set the turn flags as today; the launch line yields its prompt path; a malformed or half-written line is skipped; a missing subagents directory is zero forks; a missing projects root is no sessions.
- Claude pricing scenarios: one-hour and five-minute cache writes priced separately; Haiku 5.5 above 100K prompt tokens priced on its long card; an unknown model is unpriced; a zero-usage message costs nothing and is never unpriced.
- `SessionUsage` returns the same readings as before for the existing fixtures, id-less lines included.
- tokencount over a fake read set: sessions from two engines on one worktree are both counted; one engine with no sessions beside one with sessions counts the latter; a message key repeated across transcripts of one engine counts once; an engine without pricing makes its messages unpriced; an engine without the locator is named in the header; a named run with no sessions from any engine refuses; a locator error or an unreadable located session fails the count naming it; a session vanished after locating fails the count naming it.
- The default selection over a fake read set counts the -last most recently changed full loom runs, ranked by the locator's last-modified times, and skips a finished slug with no located session.
- The offline read set's capability values do not type-assert to `shuttleengine.Engine`.
- The time-bound cutoff leaves out a session whose earliest user or assistant entry precedes the bound, and one with no such entry, and `-no-cutoff` counts both.
- Calibration uses the instruction-file names of the Webster session's engine, read at the base commit; an absent file adds nothing.

## Notes for the plan writer

- The orch counting in `tools/tokencount/orch.go` reads the prime's sessions through the same locator; its sub-agent run attribution reads the neutral user texts.
