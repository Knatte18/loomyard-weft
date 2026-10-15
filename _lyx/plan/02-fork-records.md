# Card 2 — fork-records

**Create:**
- `tools/forkaudit/extract.jq`
- `tools/forkaudit/report.jq`
- `tools/forkaudit/testdata/fork-file.jsonl`
- `tools/forkaudit/testdata/fork-file-noedit.jsonl`
- `tools/forkaudit/testdata/fork-raf.jsonl`
- `tools/forkaudit/testdata/fork-inline.jsonl`
- `tools/forkaudit/testdata/fork-none.jsonl`
- `tools/forkaudit/testdata/fork-unknown.jsonl`
- `tools/forkaudit/testdata/not-webster.jsonl`
- `tools/forkaudit/testdata/no-prompt-read.jsonl`
- `tools/forkaudit/testdata/other-prompt.jsonl`
- `plan:tools/forkaudit#forkFixture` -> `type forkFixture struct`
- `plan:tools/forkaudit#placeForks` -> `func placeForks(t *testing.T, projects, hub string, forks []forkFixture)`

**Edit:**
- `tools/forkaudit/forkaudit.sh`
- `plan:tools/forkaudit#TestForkauditRefusals`
- `plan:tools/forkaudit#TestForkauditReport`

**Uses:**
- `docs/research/glyph-audit-2026-10-15/extract-pf.jq`
- `docs/research/glyph-audit-2026-10-15/run-extract.sh`
- `docs/research/glyph-audit-2026-10-15/perfork-pf.jq`
- `docs/research/glyph-audit-2026-10-15/report.md`

**Intent:**
Find fork transcripts, extract one record per fork, classify its arm from the prompt, and report the samples table and the per-fork table (decisions D2's discovery, D3's marker table, D4, and D5's header counts, samples and per-fork table).

`forkaudit.sh`:
- Candidates are the `*/subagents/agent-*.jsonl` files under the project directories card 1 lists; under `--since` only files with `-newermt <since>` and, with `--until`, `! -newermt <until>`; under `--runs` every such file.
  The candidates are prefiltered with `grep -l '# Webster implementer job'`, and each survivor goes through `extract.jq`.
  A candidate that cannot be read makes `grep` or `jq` fail; the script exits 1 naming the file, and card 1's trap leaves `--out` intact.
- Each transcript is read line by line (`jq -R` with `fromjson?`), so a line that does not parse is dropped; the script counts dropped lines per kept fork and in total.
- The extractor receives the project directory's run name (the directory name with the encoded hub and `-` removed) and the transcript's path relative to `--projects`, and writes one JSON record per kept fork to `records.jsonl` in the work directory.
- The header gains: candidates found, transcripts holding the heading, forks kept, and transcript lines dropped as unparsable.

`extract.jq` is `extract-pf.jq` ported:
- A fork is kept only when its first tool call (Skill and Agent calls excluded) is a Read of a Webster prompt file whose result holds `# Webster implementer job`, exactly as `extract-pf.jq` decides; anything else yields no record.
- The marker table is one array at the head of the file, checked in order, one row per arm: `file` (prompt holds the `## Start here` heading), `raf` (inline `## Kick-start pack` plus the read-all-first phrase), `inline` (inline `## Kick-start pack` without it), `none` (no pack heading and no pack-file read); a fork matching no row is `unknown`.
  A new layout adds one row.
- `cls`, `hit`, `segs`, `rtext` and the record fields are carried over unchanged, so numbers stay comparable with the kept audits: arm, run, session and agent id (from the path), start time, cards, pack opened, pack turn, pack before first edit, resolved and unresolved entries, spans and span files, the strict and file-level span reach before the first edit and in the first work turn, first-turn parallelism and its classes, calls before the first edit raw and net, exploration calls with their reads and greps, whole and partial reads, greps, member greps and quarry lookups total and before the first edit, turns, output tokens, fresh input tokens, cache reads, edits and writes and those to files with no span, and dropped lines.
  The `proj` field comes from the run name passed in, not from a hard-coded path prefix.

`report.jq` renders the report body from the slurped records; the script appends it to the header:
- Samples table: per arm, forks, runs and cards; an arm of the marker table with no forks is left out of the table and named once in a sentence under it.
- Per-fork table, one row per fork with the D4 fields `perfork-pf.jq` printed plus arm, run, agent id and dropped lines.
- Rows and arms sort by arm (marker-table order, then any other arm name alphabetically), then run, then start time, then agent id, so two runs over the same input differ only in the `Generated:` line.

Fixtures, each a synthetic Claude Code subagent transcript (one JSON object per line, `type` `assistant` or `user`, `message.id`, `tool_use` and `tool_result` content, `usage`, `timestamp`), small enough to compute by hand.
Every absolute file path inside a fixture is spelled with the placeholder `@WORKTREE@` for the fork's worktree, so card 5 can strip the real `<hub>/<run>/` prefix.
Each fixture's first-line timestamp is fixed and at least 7 seconds from every other fixture's, so a batch resolved for one fixture's start never matches another's:
- `fork-file.jsonl`: prompt with `## Start here` and two card lines; reads `<batch>-pack.md` in turn 2 alongside one other call; the pack lists span entries in a Go file `pkg/a.go` and one span in `docs/b.md`, laid out for card 5's classes: an entry whose signature text sits inside its span, one whose span holds a declaration of the name under different text, one whose declaration sits elsewhere in the file, one whose name the file lacks, one in a file absent from the start tree (`pkg/gone.go`), and the `.md` span; reads that reach some spans before the first edit; a Read of a plan card under `@WORKTREE@/_lyx/plan/` and a Read of an absolute path outside the worktree; edits, one to a file the pack gives no span for (`pkg/extra.go`), a Write of a new file (`pkg/new.go`), a `git commit`, then exploration calls including a `lyx quarry` call.
- `fork-file-noedit.jsonl`: same arm, opens the pack, reads `pkg/other.go`, which the pack gives no span for, and never edits.
- `fork-raf.jsonl`: an inline pack with the read-all-first phrase whose entries carry spans in `pkg/a.go`, the inline entry source card 5 checks.
- `fork-inline.jsonl` (inline pack whose entries carry no spans), `fork-none.jsonl` (no pack, no edit), `fork-unknown.jsonl` (reads a pack file with no `## Start here` heading, and its prompt lists no card, so it has zero cards).
- `not-webster.jsonl` lacks the heading; `no-prompt-read.jsonl` holds the heading but its first tool call is a Grep; `other-prompt.jsonl`'s first tool call reads a prompt file whose result lacks the heading, which appears only in a later message, so the prefilter passes it and `extract.jq` drops it.

Tests:
- `placeForks` copies fixtures into `<projects>/<encoded hub>-<run>/<session>/subagents/agent-<id>.jsonl`, encoding the hub like the script and replacing `@WORKTREE@` with `<hub>/<run>`, optionally appends a half-written JSON line, and sets each file's modification time to its fixture's fixed time with `os.Chtimes`.
- `TestForkauditReport` gains rows over one placement: runs `alpha` (file, file-noedit, raf, not-webster, no-prompt-read, other-prompt) and `alpha-two` (inline, none, unknown, and the raf fixture again with a truncated last line).
  `--since` over the fixtures' date keeps seven forks, reports each arm's fork, run and card counts and the header's candidate, kept and dropped counts (one dropped line, the truncated fork kept).
  It asserts `fork-file`'s per-fork row field by field against values computed by hand from the fixture (pack opened, pack turn, spans reached before the first edit and in the first work turn, first-turn parallelism, calls before the first edit, edits to files with no span).
  It runs the script twice with the same arguments, `--out` and `--work`, and asserts that the second run replaced the first report and that the two reports are identical once the `Generated:` line is removed.
  `--runs alpha` keeps only alpha's forks and names `inline`, `none` and `unknown` as arms with no forks.
  `--since` with an `--until` between two fixtures' modification times keeps only the forks modified before the bound.
- `TestForkauditRefusals` gains a row: an existing report at `--out` and a Webster candidate made unreadable with `chmod 000` exits 1 naming the file, leaves the report's bytes unchanged and leaves no other file in `--out`'s directory; it skips when `os.Geteuid() == 0`.

**ImpactSummary:** confined to `tools/forkaudit`: the script gains discovery and extraction, and the two existing table tests gain rows; no new top-level test.

**Verify:**
lyx gate test --tags integration ./tools/forkaudit
