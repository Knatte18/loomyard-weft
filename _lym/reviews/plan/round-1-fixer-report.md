# Round 1 fixer-report — plan

Every finding's premise held against the code at HEAD; every finding is fixed.
`lym loom validate-plan` reports `ok: true` with informational findings only, after the fixes and after the self-check's changes.

## Changes

- **F1** (BLOCKING) — fixed, card 8 half fixed differently.
  - Card 7: dropped the Create glyph `plan:internal/adoptcli#TestJudgeStanding` and its Tests bullet; the `TestRunWorker` row "Other subjects are refused before any step" now asserts the refusal names the recorded subjects, the step it stopped at, the re-run and `lym adopt --discard`.
  - Card 8: dropped the Create glyph `plan:internal/adoptcli#TestRunsInOrch` and its Tests bullet; its numbered-orch, empty-name and unparsable-name cases are `TestRunAdopt`'s strand rows.
  - Not made as proposed: cutting `TestRunAdopt`'s stopped and interrupted rows and the cap half of its last row.
    The proposal's premise that `TestFollow` pins them is false: `follow` returns `(adoptrecord.Record, error)` and never an exit code, and `runAdopt` step 3 owns the end-to-exit mapping and the `followCap(KeepWarm, strand)` wiring.
    With the cut, a `runAdopt` that prints `stopped` through `output.Ok` and exits 0, or always passes cap 0, passes every row (`_lym/plan/lens-review.lens-row-validity.md:30-32` made the same point).
    The stopped and interrupted rows are folded into one mapping row with `done`, and the cap half is kept as the cap-wiring half of the strand row.
  - Card 6: `TestPauseEntries` is dropped too, the same `PATTERN-test-speed` defect: its rows repeat `TestAdopt`'s pause rows and `TestRunWorker`'s listed-slug row.
    The one row only it held, an adopted beam with no entry left, became a `TestAdopt` row, and the conversion row now asserts the kept pause time.
- **F2** (MEDIUM) — fixed as proposed, refined by the self-check.
  Card 6 defines a step's outcome part: each step writes into a fresh outcome and fresh per-slug outcomes of its own, which are its part, holding the top-level fields it wrote, a `slugs` object of only the `slugOutcome` fields it set, and list fields of only its own items.
  Card 7 defines `envelopeOf`'s merge (later scalars replace, `slugs` per slug and field, lists concatenate, slugs rendered in `collect`'s order), and its two-process row also asserts the earlier reconcile warnings survive.
- **F3** (MEDIUM) — fixed as proposed, extended by the self-check.
  Card 8: with a nil `exited`, a record of another pid ends the follow with an error naming `lym beam status` and the re-run; an absent record ends it too, with the no-record error, so a `--discard` during a follow-verdict follow cannot loop forever.
  `TestFollow` gains the row for each.
- **F4** (MEDIUM) — fixed as proposed, own wording for the reaching path.
  Card 3: `WorkerAlive` reads `proc.IsRunning`, so an exited, unreaped worker reads gone.
  Card 7: the wiring-guard claim is gone; the follow verdict is a transient refusal naming `lym beam status` and `lym adopt --discard`, written before any step and writing nothing, and `TestRunWorker` gains a row reaching it with the test process's own pid and ticks.
  The reaching path is stated as a record whose worker reads alive while this worker holds the lock, as for a record without ticks whose pid was reused; the zombie path no longer reaches it once `IsRunning` reads a zombie gone (`internal/proc/proc_linux.go:31-37`).
- **F5** (MEDIUM) — fixed as proposed.
  Card 8's Long names the step order in one line, the hidden worker and the follow, `--no-orch` and `--discard` in one line each, and points to the package doc, staying under `maxLongBytes`.
  Card 6's step-list rewrite is the same one-line form.
  Card 10's doc.go list gains the end envelope, which the Long now points to.
- **F6** (MEDIUM) — fixed, the overview half fixed differently.
  Card 4 edits `fixReadError`, `fixFilingError` and `battencli#OwnedElsewhereError.Error` with the proposed texts, and its ImpactSummary names them; no test asserts the old texts (grep of `_test.go` files).
  Card 1 edits `dobbyengine#planPass` and rewords its hold-caller sentence.
  Not made as proposed: rewriting the whole `docs/overview.md` adopt entry on card 4.
  Card 4 lands before cards 6 to 8 build the worker, record, step order and pause, so the overview would describe behaviour the code lacks at card 4's landing.
  Card 4 drops the green-end sentences; card 10, which already rewrites the adopt docs over cards 4 to 8, now edits `docs/overview.md` and rewrites the entry whole with the proposed content.
- **F7** (MEDIUM) — fixed differently.
  The proposal put every change on card 7, but card 6 is where the adopt back starts returning the held-back error and where `PauseLifted` is deleted (F8), so card 6's gate would meet both.
  Card 6's bullet: the adopt back's error naming `--keep-local` and `--drop-local` is read in place of failing, and the `PauseLifted` assertion goes.
  Card 7's bullet: the record stopped at `done`, the `--drop-local` call resuming from `reconcile` and ending on the pair-add refusal naming `--keep-local`, then the `--keep-local` call recording the prime's owner.
- **F8** (LOW) — fixed as proposed, extended by the self-check.
  Card 6 deletes `runningHereRun` and `callerLabel` and says `PauseLifted` (`pause_lifted`) goes with `liftOwnerPauses`; the comment over the run constants (`internal/adoptcli/adopt.go:36-37`) is reworded to owned elsewhere or paused by adopt.
- **F9** (LOW) — fixed as proposed.
  Card 3's `**Uses:**` became `**Edit:**` with `AdoptScratchDir` and `internal/fabricengine/doc.go`, an Intent sentence on the directory's new contents, and the proposed ImpactSummary.
- **F10** (LOW) — fixed differently.
  Card 6: `runSteps` runs the standing-move refusal before its first step on every run, resumes included; `stepReconcile` no longer runs it.
  Not made as proposed: running it over "the record's local runs". A fresh run's `Inputs.LocalRuns` is empty until `stepReconcile` fills it, and a restart at `reconcile` clears it, so the check would miss the runs today's code checks (`internal/adoptcli/adopt.go:238-244` reads them with `movedRuns` first).
  `runSteps` reads them with `movedRuns` over the owners read at the start, and a refusal ends the record `stopped` at `from`.
  Card 7 gains the proposed `TestRunWorker` row.
- **F11** (NIT) — fixed as proposed.
  Card 2: with `wantNoOrch` set, no `start` is among the recorded calls and `h.spawns` stays zero.

## Self-check

- fix-claims, MEDIUM: card 7's follow-verdict reaching path named the zombie window, which card 3's `IsRunning` closes — fixed as proposed (reworded to the reachable path).
- fix-consistency, LOW: card 4's whole-entry overview rewrite describes behaviour cards 6 to 8 build — fixed as proposed (moved to card 10).
- fix-proposal, MEDIUM: cutting `TestRunAdopt`'s stopped and interrupted rows leaves the exit-code mapping unpinned — fixed as proposed (one mapping row restored).
- fix-proposal, MEDIUM: cutting the cap half leaves `runAdopt`'s cap wiring unpinned — fixed as proposed (cap half restored).
- fix-proposal, MEDIUM: an absent record under a nil `exited` still loops forever — fixed as proposed (absent record ends the follow; `TestFollow` row).
- fix-proposal, LOW: the outcome-part definition did not say how a step knows which fields it set — fixed as proposed (fresh part per step).
- fix-affected-set, MEDIUM: `TestPauseEntries` is a further member of F1's redundant-test set — fixed as proposed (dropped; its one unique row moved to `TestAdopt`).
- fix-affected-set, LOW: the comment over the run constants stays stale after F8 — fixed as proposed (card 6 rewords it).

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
