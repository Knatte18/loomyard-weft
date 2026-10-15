# Decision record — glyph-audit-tool

## Goal

One repeatable command under `tools/` that measures how Webster implementer forks read their kick-start pack, per pack layout, from Claude Code transcripts and the run records.
It replaces the hand-run jq and shell pipelines of the 2026-10-13 and 2026-10-15 audits (`docs/research/glyph-audit-2026-10-13/`, `docs/research/glyph-audit-2026-10-15/`), so the next pack layout (task `kickstart-per-card`) can be compared with today's in one run.

## Scope

In:

- A new tool directory `tools/forkaudit/`: the entry script `forkaudit.sh` and the jq filters it calls, all shell and jq.
- Fork discovery from a date window or a list of runs, arm classification, the per-fork record, the per-arm report.
- Two optional report sections behind flags: pack correctness at batch start, and plan glyph use.
- Go `_test.go` files in `tools/forkaudit/` that run the script against fixtures, and their row in `cmd/lyx/testdata/test-budget.yaml`.
  The board's shell-and-jq rule covers the tool; the gate tests are Go because `lyx gate test` runs Go tests.
- `PATTERN.md`: the `PATTERN-lyxdirs-single-declarer` line admits the script's literals (Constraints).

Out:

- The two `docs/research/glyph-audit-*` directories: left byte-for-byte as the record of their runs.
- Any change to Webster, its prompt, its trace line or the pack itself.
- The 2026-10-13 audit's `base_on` and `mid` split and its `stats-base.jq` pack-on-versus-off comparison by date: the tool compares layouts by prompt content.
- A `.cmd` launcher: the script runs under bash only (WSL on Windows).
- Per-role token totals (`tools/tokencount`) and phase timing (note `report-timing`).

## Decisions

### D1 — One tool directory, shell and jq only

`tools/forkaudit/forkaudit.sh` is the entry point; the jq filters sit beside it as `.jq` files and are found relative to the script, not the caller's cwd.
Like `tools/tokencount/tokencount.sh`, it resolves the repo root as two levels above its own directory.
It writes its report to `.scratch/fork-audit.md` under the repo root by default and prints the path; `--out <path>` overrides it.
Intermediate files (fork list, per-fork records, pack-check rows) go to `.scratch/fork-audit/` under the repo root by default, kept after the run for drill-down; `--work <dir>` overrides that one directory and nothing else.
The work directory's clearing is bounded:
- the tool marks a work directory it creates or adopts with a `.forkaudit-work` marker file;
- at the start of a run it clears only a directory that is empty or holds that marker, and deletes inside it only its own named intermediate files and subdirectories, never anything else;
- it exits 1 naming `--work`, deleting nothing, when the directory is non-empty without the marker, when it equals the repo root, the hub or `--out`'s directory or is an ancestor of any of them, or when `--out` lies inside it.
  All these comparisons use canonical paths (`realpath -m`: symlinks and `..` resolved, a not-yet-existing path resolved through its nearest existing ancestor).
  If a comparison is ever wrong, the marker rule is the bound, because a directory without the marker is never cleared.
The report is written to a temporary name beside `--out` and moved into place only on success, so a failed run leaves the previous report intact.
`--out` names a file: an existing directory at `--out` exits 1 naming `--out`, writing nothing; an existing file there is replaced.
The report write reaches exactly two paths: the file named by `--out`, and one temporary file created with `mktemp` in `--out`'s directory, so it never takes the name of an existing sibling, and removed on failure.
Nothing outside `--work` is otherwise created, replaced or deleted.
`--out` is not checked against tracked or foreign files; its bound is the explicit flag and the caller's own permissions, as with a shell redirect.
One run at a time per work directory is the supported use; two concurrent runs need distinct `--out` and `--work`.
It needs `bash`, `jq`, `git`, `find`, `awk`, `mktemp`, `realpath` and GNU `date`, and exits 1 with a message naming a missing command or an unwritable `--out` or `--work`.
The script's header comment is the tool's design doc, as `tokencount.sh`'s is, since a package with only test files has no `doc.go` (`PATTERN-documentation-lifecycle`).

Rationale: the board fixes shell and jq; one fixed output path matches the existing tool launchers; kept intermediates replace the per-fork tables the research directories kept by hand; `--work` lets tests run without touching the worktree's `.scratch/`.

### D2 — Inputs: a date window or a list of runs

- `--since <date>` with optional `--until <date>` selects fork transcripts by modification time, as the 2026-10-15 `find -newermt` did.
- `--runs <name>[,<name>...]` selects forks whose project directory belongs to those worktrees (the run's slug, the worktree's directory name).
  A run name matches only the project directory whose name equals the hub's encoded path, `-` and the run name, exactly; a name that is a prefix of another run's never matches that run.
- `--projects <dir>` (default `~/.claude/projects`), `--hub <dir>` (default the repo root's parent), `--records <dir>` and `--code <dir>` (D7) override discovery paths.
- Exit 2 with usage text on: neither or both of `--since` and `--runs`; `--until` without `--since`; `--until` earlier than `--since`; an unparsable date; an empty `--runs` list; a run name holding anything but lowercase letters, digits and `-`, the slug character set, which Claude Code's directory encoding leaves unchanged; an unknown flag or a flag missing its value.
- The report lists each `--runs` name that matched no project directory, so a typo is not mistaken for an empty run.

Discovery is the 2026-10-15 pipeline, generalized from its hard-coded paths: project directories whose name is the hub path in Claude Code's encoding followed by `-`, their `*/subagents/agent-*.jsonl` files, prefiltered on the `# Webster implementer job` heading, then kept only when the transcript's first tool call reads a Webster prompt file, as `extract-pf.jq` decides.

Rationale: the board names both input forms; deriving the project-directory prefix from `--hub` removes the `/home/knatte/...` literals that made the research scripts unrunnable elsewhere; the run-name character set keeps names out of `find` patterns.

### D3 — Arm classification from the prompt, cross-checked by the trace line

The prompt decides each fork's arm, checked in this order:

| Arm | Prompt marker |
| --- | --- |
| `file` | `## Start here` heading (pack read from `<batch>-pack.md`) |
| `raf` | inline `## Kick-start pack` plus the read-all-first phrase |
| `inline` | inline `## Kick-start pack` without that phrase |
| `none` | no pack heading and no pack-file read |
| `unknown` | anything else |

The markers live in one table at the head of the extractor, so a new layout adds one row.

The begin-batch `webster: kickstart pack` trace line's `pack_layout` field holds `file`, `inline` or `none` and cannot tell `raf` from `inline`, so it is a cross-check, never an override:

- Trace `file` agrees with arm `file`, trace `inline` with `raf` or `inline`, trace `none` with `none`; any other pairing is a disagreement, listed per fork in the report.
- A trace value outside those three, on a fork whose prompt is `unknown`, makes the fork's arm `trace:<value>`, so a new layout is measured as its own arm even before its marker row exists and never merges with a prompt-classified arm.
  This holds only when the value matches `[a-z0-9_-]+` and is not one of the marker table's arm names; any other value leaves the fork `unknown` and is listed once in the samples section as an unrecognized trace value.
- The join needs the fork's worktree and batch: the worktree is the one whose `.lyx/logs/` holds the trace file, the batch comes from D7's batch resolution.
  Of several lines for one batch (a resumed begin-batch), the last one before the fork's first transcript line counts.
- With no trace file, an unresolved batch, or a records repo that is absent and not required (D7), the fork has no cross-check; the samples section counts per arm how many forks were cross-checked.

Rationale: the board asks for classification without hand-built arms, from the prompt and, where present, the trace line; trace files go with the worktree at teardown, so the prompt is the only source for landed runs.

### D4 — The per-fork record

One record per fork, ported from `extract-pf.jq`, with these fields:

- arm, trace cross-check result, project (run), session and agent ids, start time, card count;
- pack opened (yes/no), the assistant turn it was opened in, and whether before the first edit;
- resolved and unresolved pack entries, spans and span files;
- spans reached before the first edit and in the first work turn, file-level and strict (the read overlaps the span's lines);
- first-turn parallelism (tool calls in the first work turn) and its call classes;
- calls before the first edit, net of the pack-file read;
- exploration calls (calls between a commit and the next edit, net of the pack read), with their reads and greps;
- whole-file reads, partial reads, greps, member greps and `lyx quarry` lookups, total and before the first edit;
- turns, output tokens, fresh input tokens and cache reads;
- edits and writes, and those to files with no span.

The call classifier (`cls`) and span-hit rule (`hit`) of `extract-pf.jq` are carried over unchanged, so the numbers stay comparable with both kept audits.
Transcript lines are parsed one by one and a line that does not parse is dropped and counted, so a half-written last line of a fork still running costs that line, not the fork.
A fork still running is counted with what its transcript holds so far.

Rationale: these are the board's metrics; per-line parsing keeps in-flight forks measurable.

### D5 — The report

A markdown report with:

- a header: generation time, the exact arguments, discovery paths, candidates found, forks kept, transcript lines dropped as unparsable, and `--runs` names that matched nothing;
- a samples table: per arm, forks, runs and cards, and the D3 cross-check counts and disagreement list;
- a pack-use table: per arm, medians per fork and pooled values of the D4 pack metrics, and the count of forks reaching at least half their spans before the first edit;
- an arm comparison table: per arm, medians per card and pooled per card of the D4 exploration, read, turn and token metrics, and the share of forks with three or more parallel calls in the first work turn;
- the quartiles of exploration calls per card per arm;
- a per-fork table, one row per fork;
- the optional sections of D6 and D7, each only when its flag is given.

A metric a fork does not have (no edit, so nothing "before the first edit"; a pack with no spans, so no span share; arm `none`, so no pack metric) leaves that fork out of that metric's median and pooled value, and the cell shows its own sample size where it differs from the arm's fork count, or `n/a` when none remain.
An arm with no forks is left out of the tables and named once in the samples section; zero forks in total yields a report that says so, and exit 0.
Rows sort by arm, then project, then start time, then agent id, so two runs over the same input give identical reports apart from the generation-time line.

Rationale: the board asks for medians and pooled values per card with sample sizes per arm; the layout follows the 2026-10-15 `report.md` tables.

### D6 — Optional section: plan glyph use (`--plan`)

For each run with forks in the report, the plan cards are read from the records repo at the weft commit that holds that run's `_lyx/webster/state.json`, and the references under `Edit`, `Create`, `Move`, `Delete`, `Rename` and `Uses` are counted per card by kind: glyph, `plan:` handle, bare file, bare directory, as `plan-refs.sh` classified them.
The plan writer of a run is its top-level session (not a subagent) with the most writes of `_lyx/plan/NN-*.md` cards; its quarry calls, greps and reads are counted, as `plan-writer.jq` did.
The section reports per arm: medians per card of each reference kind, and the plan writers' quarry calls against greps and reads.
A run whose weft commit or plan-writer session is not found gets one line saying which, and counts nowhere.

Rationale: the board lists plan glyph use as an optional section; finding the weft commit through `state.json` matches the 2026-10-15 method for both arms.

### D7 — Optional section: pack correctness at batch start (`--pack-check`)

- Batch resolution: for each fork, the history of `_lyx/webster/state.json` on the fork's own run's refs is searched for the batch whose `spawnedAt` is 1 to 6 seconds before the fork's first transcript line, giving batch number, cards and `startSha`, as `resolve-batches.sh` did.
  The refs are the records branch `<run>-weft`, local or under `origin/`, and the tags `archive/<run>/*`, where `<run>` is the fork's project run name from D2; a match outside those refs is not used, so a concurrent run's batch spawned in the same window never resolves.
  It runs when `--pack-check` or `--plan` is given, or when a trace file exists for the fork's worktree.
- Every span entry of the fork's pack is checked against the file at `startSha` in the code repo, classed `correct` (the entry's signature text lies inside the span), `correct-decl` (a declaration of the name lies inside it), `stale` (found elsewhere in the file, with the line shift), `wrong` (file present, name absent), `nofile` (file absent at `startSha`) or `notree` (batch unresolved or `startSha` not in the code repo).
- The declaration rule is Go-only; a span in a non-`.go` file gets the signature-text check alone and, failing it, the class `unchecked` instead of `wrong`.
- Files a fork edited or read that its pack gave no span for are listed with whether they existed at `startSha`, as `missed-files.sh` did.
- The section reports per arm: counts per class, pooled and per fork, and the missed-file counts split by existed-at-start.
- Default paths: `--code` is the repo root, which shares the hub's object store with every pair, so a pair's `startSha` resolves there; `--records` is the repo root's weft sibling `<repo root>-weft`, whose store's `git log --all` sees every pair's records branch and `archive/<slug>/*` tag.
  This is a shell simplification of what `fabricengine.RecordsWorktree` resolves for `tools/tokencount`.
- A missing `--code` directory exits 1, naming `--code`, under `--pack-check`; a missing `--records` directory exits 1, naming `--records`, under `--pack-check` or `--plan`; without either flag, a missing records repo only disables the D3 cross-check and the header says so.
- The 2026-10-15 negative control (packs checked against another batch's tree) is not a mode of the tool.

Rationale: the board lists pack correctness against the batch's start tree; the negative control was a one-time check of the checker.

### D8 — Tests: Go integration tests running the script on fixtures

- `tools/forkaudit/` holds `_test.go` files and no other Go files.
- A `testmain_test.go`, compiled under every tag set, calls `gitkit.HermeticGitEnv()` and `os.Exit(tmuxkit.Main(m))`, the shape of `tools/tokencount/testmain_test.go`, so the script's own `git` children inherit the hermetic env.
- The tests that run `forkaudit.sh` carry `//go:build integration`, since they spawn bash, jq and git.
- They run on synthetic fixtures under `testdata/`: a projects directory with one fork transcript per arm, a transcript with a truncated last line, a non-Webster subagent transcript, a plan-writer session and a trace log; for `--pack-check` and `--plan`, a small code repo and records repo built in the test.
- Every test passes `t.TempDir()` paths as `--out` and `--work`, so tests run in parallel and never touch the worktree's `.scratch/`.
- They assert on report content and exit codes, skip on Windows, where bash is not assumed, and fail with a message naming `jq` when it is missing elsewhere.
- The package gets its row in `cmd/lyx/testdata/test-budget.yaml`.

Rationale: `lyx gate test --tags integration ./...` then exercises the tool like any other code under `tools/`, as CLAUDE.md requires; synthetic fixtures keep the tests independent of real transcripts.

### Added after Discussion (parent, 2026-10-15): Shorter PATTERN-lyxdirs-single-declarer line to stay under the headroom

Decision: The PATTERN.md entry line for PATTERN-lyxdirs-single-declarer reads exactly: - `PATTERN-lyxdirs-single-declarer` — Naming `_lyx` or `.lyx` in a path: only `internal/lyxdirs` declares them, bar one variable block heading a `tools/` script. This replaces the wording prescribed in Constraints; the rest of that constraint (literals once, in a variable block at the script head) stands.

Rationale: The prescribed wording grows PATTERN.md to 15681 bytes, past the 15564-byte headroom line, so the card gate refuses it. The shorter line keeps the same rule at net -2 bytes; cutting other PATTERN entries would widen scope.

## Constraints

- `PATTERN-lyxdirs-single-declarer`: the script spells `_lyx/webster/state.json`, `_lyx/plan/` and `.lyx/logs` because it reads run records and worktree logs, and the literals sit once, in a variable block at the script's head.
  The task amends that entry's line in `PATTERN.md`, in the same commit as the script, to read: "Naming `_lyx` or `.lyx` in a Go path: only `internal/lyxdirs` declares those literals, and no other Go production file spells them; a shell tool under `tools/` that cannot import it spells each literal once, in a variable block at the script's head."
  The `-weft` suffix it spells departs in the same way from `internal/weftname`'s single-declarer convention, which is not a PATTERN entry.
- Git is read-only: `log`, `show`, `ls-tree`, `cat-file` only (`PATTERN-fabric-git` exempts code outside the binary and read-only verbs).
- The script never calls `git rev-parse --show-toplevel` and writes nothing to a system temp directory.
- `PATTERN-test-speed`: integration tier for script-running tests; no tmux, no LLM.
- `PATTERN-test-isolation`: the `TestMain` shape in D8.
- A task that adds a pack layout adds its prompt marker row to `tools/forkaudit` in the same task.
- No Python anywhere, in the tool or its tests.

## Auto-mode assumptions

- The tool is named `forkaudit`: it audits fork reads, and glyph use is one optional section.
- `jq` becomes a requirement for `lyx gate test --tags integration ./...` on non-Windows hosts.

## Open risks

- `kickstart-per-card` delivers each card's spans at card start through a tool call, not in the prompt or `<batch>-pack.md`, which are the span sources D4 parses.
  Until that task adds its marker row and a span source, its forks are their own arm through D3's trace rule and get every metric except span reach; the comparison with today's layout then rests on exploration calls, reads, turns and tokens per card.
- The batch join by `spawnedAt` within 6 seconds is heuristic; a slow fork start leaves a batch unresolved (`notree`), which the report counts.
- Transcripts and pair logs age out (Claude Code's retention, the logger's 14-day sweep), so a re-run over old dates finds fewer forks than the original audit did.
- A resumed session's transcript gets a later modification time, so `--since`/`--until` can move a fork between windows.
- Claude Code's transcript and project-directory encoding are not a stable interface; a format change breaks discovery or extraction, visible only as a drop in kept forks.

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - appended-records-path-breaks-identical-report-assertion (class consistency, severity MEDIUM, open in rounds 2)
  - filters-spell-directory-roots-outside-variable-block (class consistency, severity BLOCKING, open in rounds 2)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- `forkaudit.sh --since <date>` over fixtures with one fork per arm reports each arm with its fork, run and card counts, and the medians of D5.
- `--runs <slug>` keeps only that run's forks; a `--runs` name with no project directory is listed in the header.
- Each invalid input of D2 exits 2 with usage; a missing `jq` or an unwritable `--out` exits 1 naming it; an `--out` naming an existing directory exits 1 naming `--out`.
- A subagent transcript without the Webster heading, or whose first tool call reads no prompt file, is not a fork.
- A fork transcript with a truncated last line is kept, and the dropped line is counted in the header.
- Zero matching forks: a report saying so, exit 0.
- An arm with no forks is absent from the tables and named in the samples section.
- A fork with no edit, and a pack with no spans, are left out of the metrics they lack, with the per-cell sample size shown.
- A prompt matching no marker is classed `unknown`; with a trace value outside `file`, `inline` and `none`, the fork's arm is `trace:<value>`; a trace value of `raf`, and one holding `|`, leave the fork `unknown` and are listed as unrecognized trace values.
- A `raf` fork with trace `inline` counts as agreeing; a `file` fork with trace `inline` is listed as a disagreement and keeps arm `file`.
- Two trace lines for one batch: the last one before the fork's start is used.
- A records fixture with two runs whose batches share a `spawnedAt` resolves each fork to its own run's batch.
- A fork whose batch does not resolve gets no cross-check and, under `--pack-check`, spans classed `notree`.
- Pack opened, its turn, spans reached before the first edit and in the first work turn, first-turn parallelism, calls before the first edit and edits to files with no span match hand-computed values for a fixture fork.
- `--pack-check` classes fixture spans as `correct`, `correct-decl`, `stale`, `wrong`, `nofile`, `notree` and `unchecked`, and lists a missed file with its existed-at-start value.
- `--plan` counts glyphs, `plan:` handles, bare files and bare directories per card for a fixture plan, picks the session with the most card writes as plan writer, reports its quarry calls, greps and reads, and gives a run with no plan-writer session its one line.
- A missing `--records` directory exits 1 naming `--records` under `--pack-check`; `--plan` without a `--code` directory succeeds; without `--pack-check` and `--plan`, the run succeeds with no cross-check and the header says so.
- A failed run leaves an existing report at `--out` unchanged, and leaves no temporary file beside `--out`.
- A non-empty `--work` without the `.forkaudit-work` marker, and a `--work` holding `--out`, each exit 1 naming `--work` and delete nothing; an unwritable `--work` exits 1 naming it.
- A `--work` given as a symlink to the repo root, and one spelled with `..` to reach `--out`'s directory, each exit 1 naming `--work` and delete nothing.
- Two runs over the same fixtures produce identical reports apart from the generation-time line.
- `docs/research/glyph-audit-2026-10-13/` and `docs/research/glyph-audit-2026-10-15/` are unchanged.

## Notes for the plan writer

- Report tables port from `stats-pf.jq`, `perfork-pf.jq` and `quartiles.jq`; pack entries and touched files from `pack-entries.jq` and `touched-files.jq`; plan counts from `plan-stats.jq`, all in `docs/research/glyph-audit-2026-10-15/`, whose `COMMANDS.md` gives the order.
- The trace line is logged in `internal/websterengine/beginbatch.go`; the trace file format and its header record are in `internal/logger`.
