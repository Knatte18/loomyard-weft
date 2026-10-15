# Card 4 — forkaudit-card-arm

**Create:**
- `tools/forkaudit/testdata/fork-card.jsonl`
- `plan:tools/forkaudit#cardPackLine` -> `func cardPackLine(start time.Time, after time.Duration) string`

**Edit:**
- `tools/forkaudit/extract.jq`
- `tools/forkaudit/pack-entries.jq`
- `tools/forkaudit/forkaudit.sh`
- `tools/forkaudit/report.jq`
- `tools/forkaudit#traceCase`
- `tools/forkaudit#mainForks`
- `tools/forkaudit#TestForkauditReport`
- `tools/forkaudit#TestForkauditTraceCrossCheck`
- `tools/forkaudit#TestForkauditPackCheck`

**Intent:**
Teach the fork read audit the per-card layout, as the "Added after Discussion" entry puts in scope: a `card` arm told by the prompt and the `pack_layout=card` trace value, its pack spans read from the `lyx loom card-pack` tool results, and the `webster: card pack` trace lines as their cross-check.
No follow-up board task is filed for it.

- `extract.jq`: the marker table gains `{arm: "card", trace: ["card"], when: {cardpack: true}}` as its first row, ahead of the file row, since a per-card prompt also ends with `## Start here`.
  The new feature `cardpack` is whether the prompt holds `lyx loom card-pack`, and `armof` is passed it with the other features.
  The fork's card-pack calls are its Bash tool uses whose command holds `lyx loom card-pack` and whose result holds a `### ` subsection heading, so a refused call, whose result is a failure envelope, counts as no call and adds no pack text.
  Their results, joined in call order, are the pack text a fork of the card arm is parsed from, in place of a pack file or an inline section.
  The pack-text parser accepts a line with or without the Read tool's line-number prefix, since a Bash result carries none; Read results parse as before.
  The record gains `card_pack_calls`, the count of those calls, and `end`, the last transcript timestamp.
- `pack-entries.jq`: when the fork read no pack file and its prompt carries no inline pack, the entries come from the joined card-pack results by the same rule, so `--pack-check` classes the per-card spans like any other.
- `forkaudit.sh`: after the cross-check loop, a separate pass over every kept fork, independent of the records repository and of batch resolution, counts the `msg="webster: card pack"` lines of its worktree's trace files logged between the fork's `start` and `end`, and writes `card-traces.tsv` (transcript path, count) for each fork whose worktree has a trace file.
  `card-traces.tsv` is a new name in `WORK_FILES`, passed to `report.jq`.
  The header comment names the card arm's pack source, the card pack count and the new intermediate file, and the marker-table sentence stays.
- `report.jq`: the per-fork table gains a `card packs` column: `<calls>/<traced>` for a fork with a `card-traces.tsv` row, `<calls>/-` for a fork with card-pack calls and no row, and `-` for a fork with neither.
  A fork whose calls and traced count both exist and differ is listed under a new `Card pack disagreements:` line list after the cross-check disagreements, as `- <agent> (<run>): card-pack calls <n>, traced <m>`; a fork with no trace file is never listed.
  Its header comment says so.
- `fork-card.jsonl`: a fork whose prompt read holds the job heading, one card line, the per-card step 1 naming `lyx loom card-pack`, the sandbox sentence and a closing `## Start here`;
  then a refused Bash `lyx loom card-pack x` whose result is a failure envelope;
  then a Bash `lyx loom card-pack 1` whose result is the plain card subsection with one `pkg#Correct` entry spanning `pkg/a.go:3-6` and one unresolved ref;
  then a Read of that span by `offset` and `limit`, an Edit of `pkg/a.go`, the span's own file, and a commit.
  Its last line is stamped at least 3 seconds after its first.

Tests, rows in existing tests only, so `tools/forkaudit` stays within its row in `cmd/lyx/testdata/test-budget.yaml`:
- `cardPackLine` renders one `webster: card pack` trace line (`card=1 resolved=1 unresolved=1`) stamped `after` past start, in `traceLine`'s time format; `traceCase` gains `cardPacks int`, and `TestForkauditTraceCrossCheck`'s loop appends that many lines stamped 1 second after the fork's start to the case's trace file.
- `TestForkauditTraceCrossCheck` gains rows: `fork-card` with trace `card` and one card pack line wants arm `card`, trace `card`, cross-check `agrees`, `card packs` `1/1`, `spans` `1` and `strict before edit` `1`, which pins the prefix-less parse and the refused call counting as none;
  `fork-card` with trace `file` wants arm `card` and `disagrees`;
  `fork-card` with trace `card` and no card pack line wants `card packs` `1/0` and the report line `- agent-<run> (<run>): card-pack calls 1, traced 0`;
  `fork-card` with `noTraceFile` wants `card packs` `1/-` and no card pack disagreement line;
  `fork-card` with a `spawnLead` of 20 seconds, so no batch resolves, and one card pack line wants batch `-` and `card packs` `1/1`.
  The existing "agreeing layouts" `fork-file` row also wants `card packs` `-`, so the card row's precedence does not pull a file-layout fork into the card arm.
- `mainForks` places `fork-card` in run `alpha-two`, so every arm has a fork: `TestForkauditReport`'s "since over the fixtures" row wants one more candidate, transcript holding the job heading and kept fork in its header lines, and `wantSamples` gains `card` with one fork, one run and one card;
  its `--runs alpha` row wants "Arms with no forks: card, inline, none, unknown.", since absent arms are named in marker-table order;
  and every other row over `mainForks` takes the same count change.
- `TestForkauditPackCheck` places `fork-card` in run `alpha` against a batch spawned 3 seconds before its start and wants its span classed `correct` and its unresolved ref counted in no class, read from the card-pack result, with `wantMissed` and the file arm's missed-file counts unchanged, since its Edit touches only the span's file.

**ImpactSummary:** the audit gains a card arm, a card pack column and an intermediate file; the file, raf, inline and none arms classify and count exactly as before, which the existing rows keep pinning, and the report test's fixture set grows by one fork.

**Commit:** 4: tools/forkaudit: per-card arm read from card-pack results
