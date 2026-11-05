# Lens review — drift-rework plan

Every lens fork returned a report; each is saved beside this file as `lens-review.lens-<lens>.md`.
Card numbers below are the numbering after the review: the `lym drift check` card moved from 12 to 14 (row-validity 6, below), so the old cards 13 and 14 are now 12 and 13.
Findings shared across lenses are merged on their first line, and the other lenses are named there.

## coverage

- coverage 1 (BLOCKING) — `fixed as proposed`: card 18 lists `internal/runprofile_test#TestTemplateDocsProfile`, whose want gains `drift: {scope: off}`, pinning D3's docs-profile criterion.
- coverage 2 (MEDIUM) — `fixed as proposed`: `JudgedRange` (card 7) stores the covering entry's `why`; Drift-Verify (card 16) writes it with the judge's sentence; the verdict (card 20) renders both from `judged.json`.
- coverage 3 (MEDIUM) — `fixed as proposed`: card 15's `JudgeRequest` carries, for an added case, its current-capture file's path and, for a removed case, its baseline file's path; card 16 gains a row for an added case's request.
- coverage 4 (MEDIUM; also guard-reach 5) — `fixed as proposed`: card 19 admits `Amend` only at a blocked writer (`Darn-Write` or `Webster`), Drift-Verify or Drift-Recheck, and `Defer` only at D12's rows; `TestAmend` and `TestDeferWaive` gain rows refusing a blocked Parent-Align.
- coverage 5 (MEDIUM) — `fixed as proposed`: card 22's `readStatus` prints the diff mode and `compare_ignored: true` under `lines`; `TestCapture_OutlivesCaller` asserts it through `statusBody`.
- coverage 6 (MEDIUM) — `fixed as proposed`: card 10's gate rows cover a pass on each of `BaselineFault`, `RecordsFault` and `SealFault` with the fault in the findings, and a seal fault under `command`; card 14's test covers a missing baseline reported without failing.
- coverage 7 (MEDIUM) — `fixed as proposed`: card 16 gains rows for a never-counted capture, an edited baseline before any judge, and a missing baseline directory with no `capture.json`.
- coverage 8 (MEDIUM) — `fixed as proposed`: card 12 gains an exhausted `expectation` entry row and a respawn on the next call, and a `NewExpectationGate` sub-run (valid, missing, invalid file); card 16 gains the exhausted `judgement` row.
- coverage 9 (MEDIUM) — `fixed as proposed`: cards 12 and 16 assert their Stuck reasons open with `DecisionHaltPrefix` followed by `DeferHaltPrefix`; card 19's `TestDeferWaive` admission rows use reasons in that order.
- coverage 10 (MEDIUM; also way-forward 8) — `fixed differently`: the finding proposes no change beyond research and a row. The fact is now researched and recorded in Shared Decision `tapestry-fix-route`: a re-entered Webster has no done short-circuit (`internal/loomshed/webster.go` `Call`; `internal/websterengine/runlevel.go:389` to `StartMaster` returns early only on errors), so Merriam starts and runs the run-level `drift` entry. The route is derived from `Deps.Rows` in card 16, which drops the `FixRoute` field, and `TestDriftVerify` gains a row for tapestry's rows.
- coverage 11 (MEDIUM) — `fixed as proposed`: card 16 gains a row recording config's `diff` and patterns on a record with none and keeping them over a later `key_patterns` change; card 10 gains a row reading config's mode for a record with none; card 12 records the values too (guard-reach 4).
- coverage 12 (MEDIUM) — `fixed as proposed`: card 10's `TestCheck` gains an old-`partial` row that leaves the skip list's cases out of the diff.
- coverage 13 (MEDIUM) — `fixed as proposed`: card 12 gains a row where the seal commit carries `Drift-Baseline` when no digest commit exists yet.
- coverage 14 (MEDIUM) — `fixed as proposed`: card 17 gains a row where an all-`matches` judge counts as a pass for the unchanged-head skip; card 16 gains a row recording `Unsealed` with neither file nor seal.
- coverage 15 (MEDIUM) — `fixed as proposed`: `TestAmend` (card 19) gains rows for `--restore` with nothing failed, for none or two of the three request kinds, and for `command`-mode entries dropping case rulings; the new `TestAmend_RefusesWhatItCannotRead` (tier1) covers an unknown slug and an unreadable `--entries` file.
- coverage 16 (MEDIUM) — `fixed as proposed`: card 9's `TestCaptureCurrent` row runs a check holding the gate's slot and a second check with none on a one-slot pool, and both finish.
- coverage 17 (LOW; also forced-edits 9) — `fixed as proposed`: the parent recorded the departure with `lym loom decision add` as the `Added after Discussion` entry "Old drift chairs end through the strand table". Shared Decision `old-chairs-end-through-the-strand-table` and card 16 cite it and state its bound.
- coverage 18 (LOW) — `fixed as proposed`: rows added for each listed branch:
  - card 1: unborn weft and key-charset refusal;
  - card 3: a pattern with no `key` group;
  - card 9: exit 0 with no file;
  - card 10: `TestWebsterEntry_GateKeys` row for the Webster row's `drift` gate;
  - card 12: a closed mark with an invalid file spawns the agent;
  - card 16: resume under the recorded spawn id, and a replaced open range;
  - card 17: an amendment since stored rulings re-judges;
  - card 20: a `command`-mode verdict.
- coverage 19 (LOW; also forced-edits 7) — `disputed`, premise false: `internal/reportengine/timingspans.go:102,120` key `loomshed.NameDriftRecheck`, the row name `Drift-Recheck`, which D4 keeps. No engine name is read there, so deleting the `DriftRecheck` engine (card 23) leaves those maps correct, and card 13 already edits them for the new row.

## row-validity

- row-validity 1 (BLOCKING) — `fixed as proposed`: card 25 re-signs `internal/driftengine#statusAt` without `agentSession` and lists `TestAmend`, its other caller.
- row-validity 2 (BLOCKING) — `fixed as proposed`: card 24 edits `internal/shedrecipe#driftTestEnv` to stop filling `BaselineTable`, `VerifyTable` and `CompareAccepted`.
- row-validity 3 (MEDIUM) — `fixed as proposed`: card 7 declares the shared fake `commitStore`, `newCommitStore` and `commitStore.deps`, and cards 8, 10, 12, 16, 17, 19 and 20 name them under Uses; `TestDeferWaive` builds its `Deps` through it.
- row-validity 4 (MEDIUM) — `fixed as proposed`: `CaptureRequest` gains `InheritedSlot`, which the wiring reads from `gateslot.InheritEnv` (cards 9, 14), so the test sets a field and stays parallel.
- row-validity 5 (MEDIUM) — `fixed as proposed`: card 22 states the anchor test's seeded scope record, capture record, baseline and status past Drift-Expect, so `check` reaches its capture.
- row-validity 6 (MEDIUM) — `fixed differently`: the proposed fix was to state the fixture. Stating it was not enough: the check verb's card sat before the Drift-Expect rows (old card 12 before old card 14), and since `PastRow` treats a row absent from the recipe as not past, `check` read every row as before Drift-Expect. The card now states the fixture and also moved after the rows card with `lym loom remove-card` and `lym loom insert-card` (now card 14).
- row-validity 7 (LOW) — `fixed as proposed`: card 18's `partial` row asserts the recorded `full` only and names why the Warn is not asserted.
- row-validity 8 (LOW) — `fixed as proposed`: card 8 drops the worktree-edit row and names the edited-baseline row as the one carrying D7's criterion.
- row-validity 9 (LOW) — `fixed as proposed`: card 26 adds a `TestCaptureCurrent` row for a `{skip_file}` command, which `CaptureCurrent` returns as `Passed` false with the refusal.

## guard-reach

- guard-reach 1 (BLOCKING) — `fixed as proposed`: new Shared Decision `counted-from-the-tracked-record`. Cards 10, 12 and 16 judge a counted capture from `awaited_at` and `CheckBaseline`, never `capture.json`, and gain rows with no capture record.
- guard-reach 2 (BLOCKING; also way-forward 1) — `fixed as proposed`: Shared Decision `stop-prefixes` fixes the order, `DecisionHaltPrefix` and then `DeferHaltPrefix`; cards 8 and 12 now say it, and card 19's `Defer` reads the defer prefix past the decision prefix.
- guard-reach 3 (MEDIUM) — `fixed as proposed`: card 16 records `Unsealed` only with neither seal nor `expected.yaml`, so a file with no seal reaches `CheckSeal`'s `--discard` fault, and gains that row.
- guard-reach 4 (MEDIUM) — `fixed as proposed`: card 12's Drift-Expect records config's `diff` and `key_patterns` on a record with none, with a row.
- guard-reach 5 — merged into coverage 4.
- guard-reach 6 (LOW; also way-forward 10) — `fixed as proposed`: card 26 calls the `{skip_file}` refusal first in `RunCapture`, `CaptureCurrent` and `RunCompare`, before the slot, the lock, the running record and the backup, with rows asserting the slot is free and the baseline untouched.
- guard-reach 7 (LOW) — `fixed as proposed`: card 12's `NewExpectationGate` falls back to config's mode when the record holds none.

## way-forward

- way-forward 1 — merged into guard-reach 2.
- way-forward 2 (MEDIUM) — `fixed as proposed`: card 12's moved-head Stuck carries `DecisionHaltPrefix` and then `DeferHaltPrefix`, and its baseline head reads the scope record before `capture.json`.
- way-forward 3 (MEDIUM) — `fixed as proposed`: card 16's step 5 compares the trees with the recorded snapshot before resuming or respawning the judge, and a respawn keeps that snapshot and spawn id; a row covers a resume with the edit still present.
- way-forward 4 (MEDIUM) — `fixed as proposed`: Shared Decision `stop-prefixes` and card 16 give the judge-change and snapshot-lost Stucks `DecisionHaltPrefix`.
- way-forward 5 (MEDIUM) — `fixed as proposed`: card 9 returns a step refusal from `CaptureCurrent` as `Passed` false with the refusal as `Reason`, never as an error, so Drift-Verify's Stuck and the gate carry it; card 26 does the same for `{skip_file}`, with rows.
- way-forward 6 (MEDIUM) — `fixed differently`: admitting `lym drift defer` at a writer's capture-failure Stuck would widen D12, which admits defer at the writer only for a missing or changed baseline. Instead, card 10's `CaptureFault` findings name the way forward:
  - the writer fixes the code when the change broke the capture;
  - otherwise the stop goes to the parent, who fixes the environment the capture names and then runs `lym loom resume`.
- way-forward 7 (MEDIUM) — `fixed as proposed`: card 11's verify-fix stencil has the fixer leave a value the record or plan shows as intended and say so, so the entry stays failing and the parent amends at Webster's Stuck. Card 21's Master text says the same.
- way-forward 8 — merged into coverage 10.
- way-forward 9 (LOW) — `fixed as proposed`: card 19's `lym drift defer` takes an optional `<slug>` resolved as `amend` resolves it, with its bound stated.
- way-forward 10 — merged into guard-reach 6.
- way-forward 11 (LOW) — `fixed as proposed`: card 18's recipe headers say a run parked at an old halt naming a removed verb goes on with `lym loom resume`.

## forced-edits

- forced-edits 1 (BLOCKING) — `fixed as proposed`: card 20 lists `internal/loomcli#loomCLI.wire`, whose Describe-table closure calls `Verdict` (`internal/loomcli/wiring.go:809`), and its ImpactSummary names both callers.
- forced-edits 2 (BLOCKING) — `fixed as proposed`: cards 10, 13 and 16 edit `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml` with the recipe.
- forced-edits 3 (BLOCKING) — `fixed differently`: the finding proposed listing the shape test's table on cards 16 and 23.
  - Its premise is half false: cards 13 and 16 already list `internal/loomrecipe#wantProducerTable`, the table in `shape_test.go:83`.
  - What was missing is now planned:
    - card 16 deletes `internal/loomrecipe#driftRecheckType` when the row moves to `driftVerifyType`, since card 23's delete of the engine type would otherwise break it;
    - card 13 declares `driftExpectType` for the new row.
- forced-edits 4 (BLOCKING) — `fixed as proposed`: card 12 edits `internal/shedbuild#engineMinimalConfig` with `DriftExpect`'s `subject` and `expectation` gate.
- forced-edits 5 (MEDIUM) — `fixed as proposed`: card 13 lists `internal/loomrecipe#TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`.
- forced-edits 6 (MEDIUM) — `fixed as proposed`: card 16 lists `internal/loomrecipe#TestRouting_ProgressAt` and `internal/loomrecipe#TestDriftRows_EntryGate`.
- forced-edits 7 — merged into coverage 19.
- forced-edits 8 (LOW) — `fixed as proposed`: card 24 rewrites `parentdirective_test.go`'s header comment to name the judge seat under `judge_interactive`, listing every member of the file as `edit-test-file`'s way forward asks; the transient `drift-template-verify.md` mention goes with that stencil on the same card.
- forced-edits 9 — merged into coverage 17.
