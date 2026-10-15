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
  For a fork of the card arm, each card's spans come after the earlier cards' edits and none is read in the first work turn, so the span windows change:
  `spans_strict_pre` and `spans_file_pre` count a span as reached when it is read after the card-pack call whose result lists it and before the first Edit, Write, MultiEdit or NotebookEdit following that call,
  and `spans_strict_first` and `spans_file_first` are null.
  A span that two calls list identically counts as reached when some call listing it precedes a read of it that precedes the next edit after that call.
  For a fork of the card arm, `ex_net` and `pre_edit_net` also exclude the card-pack calls counted in `card_pack_calls` that fall inside the counted range, as the file arm's exclude its pack read;
  `ex_tools`, `tools` and the other raw columns keep them.
  Every other arm's span and net fields are computed as today.
- `pack-entries.jq`: when the fork read no pack file and its prompt carries no inline pack, the entries come from its successful card-pack calls, by the same rule.
  The first such call, the one resolved against the batch's start tree, yields rows of kinds `span` and `unres` as a pack file does;
  each later call, resolved against a tree the start sha does not hold, yields its span lines as rows of a third kind, `later`, and no `unres` rows.
  Its header comment's Columns line names kind `later` beside a sentence saying only the first card-pack call yields `span` rows.
- `forkaudit.sh`: the pack loop records `span_files["$fork_rel|$file"]=1` for rows of kind `span` and kind `later`, so a file only a later card lists is not a missed file,
  and classifies and writes `pack-check.tsv` rows for kind `span` only, so later calls are never graded.
  The header comments of `forkaudit.sh` (its pack-correctness paragraph, which also names kind `later`) and `report.jq` say the card arm's pack check grades its first call only.
  After the cross-check loop, a separate pass over every kept fork, independent of the records repository and of batch resolution, counts the `msg="webster: card pack"` lines of its worktree's trace files logged between the fork's `start` and `end`, and writes `card-traces.tsv` (transcript path, count) for each fork whose worktree has a trace file.
  `card-traces.tsv` is a new name in `WORK_FILES`, passed to `report.jq`.
  The header comment names the card arm's pack source, the card pack count and the new intermediate file, and the marker-table sentence stays.
- `report.jq`: the per-fork table gains a `card packs` column: `-` for a fork with no card-pack call and no traced card pack line (no `card-traces.tsv` row, or a row whose count is 0), `<calls>/<traced>` for a fork with a `card-traces.tsv` row and either a card-pack call or a nonzero traced count, and `<calls>/-` for a fork with card-pack calls and no row.
  A fork with a `card-traces.tsv` row whose card-pack call count differs from the row's count, a count of 0 calls included, is listed under a new `Card pack disagreements:` line list after the cross-check disagreements, as `- <agent> (<run>): card-pack calls <n>, traced <m>`; a fork with no trace file is never listed.
  The two "spans reached in the first work turn" rows yield null for a fork whose `spans_*_first` is null, so they leave the card arm out, and the per-fork table shows `-` in its `strict in first turn` cell.
  Its header comment says so, states the card arm's span window under the missing-metric rule, and names the card arm's three note lines below.
  Whenever the card arm is present, the report prints a note line directly under each of the Pack use, Arm comparison and Per-fork tables:
  - under Pack use: "Card arm: a span counts as reached when read after the card-pack call that lists it and before the next edit; the other arms count a span read before the batch's first edit, and the card arm has no first-work-turn share because that turn reads card files only."
  - under Arm comparison: "Card arm: the first work turn reads card files only, so its share of forks with 3+ parallel calls counts card-file reads, not span reads; the other arms' first work turn holds their span reads."
  - under Per-fork: "Card arm: strict before edit and file-level before edit use the window stated under the Pack use table, strict in first turn is - and first parallel counts the card-file reads."
  When `$pack_check` is on and the card arm is present, the report also prints one line directly under the `## Pack correctness at batch start` heading, after its blank line:
  "Card arm: its spans here come from its first card-pack call only, resolved at the batch's start; spans from later calls are resolved against edited trees and are not graded, so the count is below its distinct spans under Pack use."
- `fork-card.jsonl`: a fork whose prompt read holds the job heading, two card lines, the per-card step 1 naming `lyx loom card-pack`, the sandbox sentence and a closing `## Start here`;
  then a refused Bash `lyx loom card-pack x` whose result is a failure envelope;
  then a Bash `lyx loom card-pack 1` whose result is the plain card subsection with one `pkg#Correct` entry spanning `pkg/a.go:3-6` and one unresolved ref;
  then a Read of that span by `offset` and `limit`, an Edit of `pkg/a.go`, the span's own file, and a commit;
  then a Bash `lyx loom card-pack 2` whose result lists two entries, `pkg#Stale` spanning `pkg/a.go:12-14` with signature `func Stale()` and `pkg#Other` spanning `pkg/extra.go:1-1`, neither of which is read;
  then an Edit of `pkg/extra.go`.
  Its first line is stamped 2026-10-14T09:06:10.000Z, so its modification time two minutes later is past the `until` bound 2026-10-14T09:04:40Z, and its last line is at least 3 seconds after its first.

Tests, rows in existing tests only, so `tools/forkaudit` stays within its row in `cmd/lyx/testdata/test-budget.yaml`:
- `cardPackLine` renders one `webster: card pack` trace line (`card=1 resolved=1 unresolved=1`) stamped `after` past start, in `traceLine`'s time format; `traceCase` gains `cardPacks int`, and `TestForkauditTraceCrossCheck`'s loop appends that many lines stamped 1 second after the fork's start to the case's trace file.
- `TestForkauditTraceCrossCheck` gains rows: `fork-card` with trace `card` and two card pack lines wants arm `card`, trace `card`, cross-check `agrees`, `card packs` `2/2`, `spans` `3`, `strict before edit` `1`, `strict in first turn` `-`, `calls before edit` `3`, `net before edit` `2` and `net exploration` `2`, which pins the prefix-less parse, the refused call counting as none, the card arm's span window and the netting of card-pack calls:
  the fork's tool calls before its first Edit are the refused card-pack call, the successful card-pack call and the span Read, of which only the successful call is netted, and its exploration calls are those three and the second card-pack call, of which both successful calls are netted, since the Edit of `pkg/extra.go` closes the segment after the second call and adds no read;
  that row's `wantReport` holds "Card arm: a span counts as reached when read after the card-pack call that lists it", "Card arm: the first work turn reads card files only" and "Card arm: strict before edit and file-level before edit use the window";
  `fork-card` with trace `file` wants arm `card` and `disagrees`;
  `fork-card` with trace `card` and no card pack line wants `card packs` `2/0` and the report line `- agent-<run> (<run>): card-pack calls 2, traced 0`;
  `fork-card` with `noTraceFile` wants `card packs` `2/-` and no card pack disagreement line;
  `fork-card` with a `spawnLead` of 20 seconds, so no batch resolves, and two card pack lines wants batch `-` and `card packs` `2/2`.
  The existing "agreeing layouts" `fork-file` row also wants `card packs` `-`, so the card row's precedence does not pull a file-layout fork into the card arm.
- `mainForks` places `fork-card` in run `alpha-two`, so every arm has a fork: `TestForkauditReport`'s "since over the fixtures" row wants one more candidate, transcript holding the job heading and kept fork in its header lines, and `wantSamples` gains `card` with one fork, one run and two cards;
  its `--runs alpha` row wants "Arms with no forks: card, inline, none, unknown.", since absent arms are named in marker-table order, and asserts the report holds no "Card arm:" line, which covers every card arm note;
  the `--runs alpha` row keeps its three kept forks because fork-card sits in run alpha-two, and the `until` row keeps its three kept forks because fork-card's default modification time is past its bound.
- `TestForkauditPackCheck` places `fork-card` in run `alpha` against a batch spawned 3 seconds before its start and wants agent-fcard `correct` 1 and `stale` 0, its unresolved ref counted in no class, read from the first card-pack result only, with `wantMissed` and the file arm's missed-file counts unchanged, since every file it edits or reads is listed by one of its card-pack results;
  the report holds "Card arm: its spans here come from its first card-pack call only".

**ImpactSummary:** the audit gains a card arm, a card pack column and an intermediate file; the file, raf, inline and none arms classify and count exactly as before, which the existing rows keep pinning, and the report test's fixture set grows by one fork.

**Commit:** 4: tools/forkaudit: per-card arm read from card-pack results
