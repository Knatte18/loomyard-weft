# Round 3 fixer-report

All three findings fixed in `_lyx/plan/04-forkaudit-card-arm.md`.

## Changes

- **F1 (BLOCKING), fixed as proposed, plus one pin the fix moved.**
  The `pack-entries.jq` bullet now emits the first successful card-pack call's rows as kinds `span` and `unres`, and each later call's span lines as kind `later` with no `unres` rows; its Columns header line names `later`.
  The `forkaudit.sh` bullet (the two bullets for that file are merged into one) fills `span_files` from kinds `span` and `later` and classifies and writes `pack-check.tsv` rows for kind `span` only; its pack-correctness header paragraph names `later`.
  The `fork-card.jsonl` bullet's second card-pack result lists a second entry, `pkg#Other` spanning `pkg/extra.go:1-1`, followed by an Edit of `pkg/extra.go`.
  The `TestForkauditPackCheck` bullet now says `wantMissed` stays unchanged "since every file it edits or reads is listed by one of its card-pack results".
  The review said the `fork-card` pins do not move, but the new entry raises the fork's distinct spans: `extract.jq`'s `nspans` is the length of the distinct spans parsed from all calls' joined pack text (`tools/forkaudit/extract.jq:124`), so the agreeing row's `spans` pin goes from `2` to `3`.
  `calls before edit` 3, `net before edit` 2 and `net exploration` 2 stay, and the netting sentence now says the `pkg/extra.go` Edit closes the segment after the second call and adds no read.
- **F2 (MEDIUM), fixed as proposed.**
  The `report.jq` bullet prints a card arm note under each of the Pack use, Arm comparison and Per-fork tables, with the review's wording, and its header comment names the three notes.
  The agreeing `fork-card` row's `wantReport` holds the two new leads; the `--runs alpha` row's no "Card arm:" assertion now covers every note.
- **F3 (LOW), fixed as proposed.**
  The `report.jq` bullet prints the first-call-only line under `## Pack correctness at batch start` when `$pack_check` is on and the card arm is present; the `TestForkauditPackCheck` bullet asserts its lead.

No target is added, dropped or moved, and no verify command changes.

## Disputed

None.

## Deferred

None.
