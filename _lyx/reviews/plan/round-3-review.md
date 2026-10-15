---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/04-forkaudit-card-arm.md (pack-entries.jq bullet; fork-card.jsonl bullet; TestForkauditPackCheck bullet)
    summary: Leaving later card-pack calls out of pack-entries.tsv also drops their files from the missed-files check, so the card arm lists every later card's target as a file its pack gave no span for, and no fixture row reaches it
    fix: |
      In card 4's `pack-entries.jq` bullet, keep the first successful card-pack call's rows as they are (kinds `span` and `unres`), and emit each later successful call's span lines as rows of a third kind, `later`, with no `unres` rows for them.
      `forkaudit.sh`'s pack loop then records `span_files["$fork_rel|$file"]=1` for kind `span` and kind `later`, and classifies and writes `pack-check.tsv` rows for kind `span` only, so the grading still ignores later calls (the `stale 0` pin keeps holding).
      Name kind `later` in the header comments of `pack-entries.jq` (its Columns line) and `forkaudit.sh` (the pack-correctness paragraph), beside the first-call sentence.
      In the `fork-card.jsonl` bullet, make the second card-pack result (card 2) list a second entry, `pkg#Other` spanning `pkg/extra.go:1-1`, and add an Edit of `pkg/extra.go` after that call.
      The pinned cells of the `fork-card` rows do not move: calls before the first edit stay 3, and the netted exploration count stays 2 because the new Edit closes the segment after the second call, and the trailing Edit adds no read.
      In the `TestForkauditPackCheck` bullet, replace "since its Edit touches only the span's file" with "since every file it edits or reads is listed by one of its card-pack results", so `wantMissed` and the file arm's missed-file counts stay unchanged only when later calls' files count as covered.
      No target is added, dropped or moved, and no verify command changes, so `validate-plan --overlay` is not needed.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/04-forkaudit-card-arm.md (report.jq bullet, the Card arm note line); tools/forkaudit/report.jq:125 and :133-143
    summary: The card arm window note sits under the Pack use table only, so the Arm comparison row for 3+ parallel calls in the first work turn and the Per-fork before-edit and first-parallel columns print the card arm beside the others with nothing saying their window differs
    fix: |
      In card 4's `report.jq` bullet, keep the note under the Pack use table and add two note lines, each printed only when the card arm is present.
      Directly under the Arm comparison table, a line starting "Card arm: the first work turn reads card files only, so its share of forks with 3+ parallel calls counts card-file reads, not span reads; the other arms' first work turn holds their span reads."
      Directly under the Per-fork table, a line starting "Card arm: strict before edit and file-level before edit use the window stated under the Pack use table, strict in first turn is - and first parallel counts the card-file reads."
      Add both leads to the agreeing `fork-card` row's `wantReport` in `TestForkauditTraceCrossCheck`, and keep the `--runs alpha` row's assertion that the report holds no "Card arm:" line, which now covers all three notes.
      Update the `report.jq` header-comment sentence in the same bullet so it names the three notes.
      No target changes, so no overlay run is needed.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/04-forkaudit-card-arm.md (pack-entries.jq bullet, last clause); tools/forkaudit/report.jq:144-161
    summary: That the card arm's pack check covers its first card-pack call only is stated in two source header comments but not in the printed Pack correctness section, where the card arm's span count is lower than its distinct spans per fork under Pack use
    fix: |
      In card 4's `report.jq` bullet, when `$pack_check` is on and the card arm is present, print one line directly under the `## Pack correctness at batch start` heading, after its blank line: "Card arm: its spans here come from its first card-pack call only, resolved at the batch's start; spans from later calls are resolved against edited trees and are not graded, so the count is below its distinct spans under Pack use."
      Add to the `TestForkauditPackCheck` bullet that the report holds "Card arm: its spans here come from its first card-pack call only".
      Keep the header-comment sentences the bullet already prescribes.
      No target changes, so no overlay run is needed.
---

### [BLOCKING] Later card-pack calls vanish from the missed-files check

**Location:** `_lyx/plan/04-forkaudit-card-arm.md`, the `pack-entries.jq` bullet; consumed at `tools/forkaudit/forkaudit.sh:443-464`.

**Issue:** The plan emits `pack-entries.tsv` rows from the first successful card-pack call only, "so their entries are left out".
That keeps later calls out of the grading, which is right: they resolve against trees the batch's start sha does not hold, so grading them would call shifted spans stale and new files `nofile`.
The `stale 0` pin in the `TestForkauditPackCheck` bullet reaches that case.
But `forkaudit.sh` fills `span_files` from the same `entries` loop (line 451), and the missed-files pass skips only files in `span_files` (line 456).
With later calls absent, every file that only card 2 or later lists a span for, and that the fork edits or reads, lands in `missed-files.tsv` as a file "its pack gave no span for".
A batch with several cards touching different files, the normal shape of a batch, therefore gives the card arm a missed-file table that is wrong by construction, and `extract.jq`'s `edits_out` (computed from all calls' spans) disagrees with it.
The fixture does not reach this: its second call lists `pkg/a.go`, the file the first call already covers, and the `TestForkauditPackCheck` bullet explains the unchanged `wantMissed` by "its Edit touches only the span's file".
The fix keeps the grading as planned and adds a row kind that feeds only `span_files`, with a fixture edit that makes `wantMissed` red without it.

### [MEDIUM] The window note does not reach the rows that print the card arm beside the others outside the Pack use table

**Location:** `_lyx/plan/04-forkaudit-card-arm.md`, the `report.jq` bullet; `tools/forkaudit/report.jq:125` (Arm comparison, forks with 3+ parallel calls in the first work turn) and `:133-143` (Per-fork).

**Issue:** The plan prints one note "directly under the Pack use table".
Inside that table a reader sees the note: the before-the-first-edit span rows differ in window, and the two first-work-turn rows show n/a for the card arm.
The row "forks with 3+ parallel calls in the first work turn, share" sits in the Arm comparison table and is computed from `first_par` for every arm.
For the card arm `extract.jq` makes the first work turn the turn after the prompt read (no pack read, `base_turn` is the prompt turn), which D2 makes the card-file read turn, so the card arm will print a high share from reading card files while the file arm's share counts span reads in its first turn.
Nothing in the printed report says so: the note is a table away and speaks of spans only.
The Per-fork table repeats the problem in its `strict before edit`, `file-level before edit` and `first parallel` columns.
The question the round asked is whether a reader of the printed report can tell; for these rows the answer is no.

### [LOW] The first-call-only pack check is documented in source comments, not in the printed report

**Location:** `_lyx/plan/04-forkaudit-card-arm.md`, the `pack-entries.jq` bullet's last clause; `tools/forkaudit/report.jq:144-161`.

**Issue:** The plan has the header comments of `forkaudit.sh` and `report.jq` say the card arm's pack check covers its first call.
The report's reader never sees those comments.
In the Pack correctness tables the card arm's `spans` count comes from one call, while `distinct spans per fork` under Pack use counts the spans of every call, so the two numbers for one arm disagree with no explanation in the report.

## Rejected

- Focus question on the netting of card-pack calls in `ex_net` and `pre_edit_net`: not a defect.
  The `extract.jq` bullet nets the successful card-pack calls inside the counted range and leaves the raw columns alone, as the file arm's single pack read is netted.
  A card-pack Bash call has no `cls` class, so `rfull_net` and the grep, quarry and read columns need no netting.
  The `fork-card` row pins `calls before edit` 3, `net before edit` 2 and `net exploration` 2, which match the fixture arithmetic (refused call, call 1 and the span Read before the Edit; those three plus call 2 after the commit, with both successful calls netted).
- Focus question on grading later calls against the batch's start sha: the plan grades them against nothing, which is right, and the `TestForkauditPackCheck` row pins `stale 0` for the card 2 call; the remaining gap is F1.
- The first-call rule when a fork's first card-pack call is a degraded pack: it still holds a `### ` heading, so it counts as the first successful call, which matches the verb printing every glyph unresolved with exit 0.
