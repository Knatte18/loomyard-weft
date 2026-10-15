# Fixer report — plan, round 2

## Changes

- **F1** (`04-forkaudit-card-arm.md`), fixed as proposed.
  The `report.jq` bullet prints the "Card arm: …" note line under the Pack use table whenever the card arm is present, and the header-comment sentence stays.
  The `extract.jq` bullet states the rule for a span two calls list identically.
  The agreeing fork-card trace row's `wantReport` holds the note's prefix, and the `--runs alpha` row asserts the report has no "Card arm:" line.
- **F2** (`04-forkaudit-card-arm.md`), fixed as proposed.
  For the card arm, `ex_net` and `pre_edit_net` exclude the counted card-pack calls inside their range, and the raw columns keep them.
  The agreeing fork-card row wants `calls before edit` 3, `net before edit` 2 and `net exploration` 2.
  `net exploration` counts the second card-pack call, which falls in the exploration segment after the commit, and both successful calls are netted, so the value stays 2.
- **F3** (`04-forkaudit-card-arm.md`), fixed as proposed, with follow-on corrections the fix key did not name.
  The `pack-entries.jq` bullet takes entries from the first successful card-pack call only and says the `forkaudit.sh` and `report.jq` header comments state this.
  The fixture adds a `lyx loom card-pack 2` call after the commit, listing `pkg#Stale` at `pkg/a.go:12-14` with `func Stale()`, and `TestForkauditPackCheck` wants agent-fcard `correct` 1 and `stale` 0.
  The agreeing row wants `spans` 2 with `strict before edit` still 1.
  Follow-on corrections: the second successful call makes `card_pack_calls` 2, so leaving the trace rows as they were would have made the agreeing row `2/1` and listed it as a card pack disagreement.
  The agreeing and spawnLead rows therefore get two card pack lines and want `2/2`, the no-line row wants `2/0` and `card-pack calls 2, traced 0`, and the noTraceFile row wants `2/-`.
  The fixture prompt holds two card lines so a `card-pack 2` call has a card 2, and `wantSamples` gives `card` two cards.
- **F4** (`03-per-card-layout.md`), fixed as proposed.
  The four fragment files are on the Edit list after `webster-fragment-pack-read.md`, and the Intent gives their new banner conditions, with the text after each banner kept byte-identical.
- **F5** (`02-card-pack-verb.md`), fixed as proposed: the pre-run sentence is replaced with the review's text.
- **F6** (`01-per-card-switch.md`), fixed as proposed.
  The `Config` doc-comment and `template.yaml` comment descriptions say the per-card layout applies to a hub run, that a standalone run keeps the layout `kickstart_pack_file` picks, and that every begin-batch reads the key afresh.
- **F7** (`04-forkaudit-card-arm.md`), fixed as proposed.
  The "every other row" sentence is replaced with the `--runs alpha` and `until` reasoning, and the fixture's first line is stamped 2026-10-14T09:06:10.000Z.
- **F8** (`04-forkaudit-card-arm.md`), fixed as proposed: the disagreement rule now covers a count of 0 calls.

`lyx loom validate-plan` over the edited plan returns `ok: true` with informational `create-new-unit` findings only.

## Disputed

None.

## Deferred

None.
