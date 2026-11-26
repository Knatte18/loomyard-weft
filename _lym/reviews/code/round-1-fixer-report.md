---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "ValidateDescription (internal/summaryparser/description.go) refuses a Blast radius line of more than one word, so the proposed list there would fail the gate; the line reads shuttle and a paragraph under it names the rest."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: "resumableAfterInterruption also accepts died records and running records with no waiter, so keeping all it accepts would keep every such diagnosis directory until a start of the same role and outputs set it aside; the sweep keeps only a stopped record it accepts, which is the one goneStrandReason reads."
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
---

## What changed

- **F1:** `awaitFixerStrand` takes a `gatefixer.Deps`; `FixVerifyAwait` and `FixPlanAwait` set `ShuttleCfg` and `AnchorRoot`. New `TestFixVerifyAwait` row for an interrupted run.
- **F2:** `waitOnly` passes `afterFixer` as `replace`, so `Final` replaces an interrupted fixer. Another entry that fails over one leaves its record live.
  Added beyond the proposal: that record is marked known to the wrap (`spawned` renamed `known`),
  because `Expired` would otherwise report it while the writer reworks and finalize the writer mid-turn. Docs and test rows updated.
- **F3:** a judge run error with no recorded verdict fails the call, wrapping the error. The `RunError` degrade row is removed, with a `Loosened-Check` trailer.
- **F4:** `_lym/landing/summary.md` rewritten as the disposition says, and its prose brought in line with F2, F3 and F7. Not committed.
- **F5, F6:** the interrupted note no longer says no replacement ran, and names a replacement's spawn error when there is one.
- **F7:** `sweepOrphans` keeps a stopped, interrupted, resumable record.
- **F8:** the replacement rows assert `Expired()` is false while the replacement is waited on.
- **F9:** the box disagreement warns once per strand GUID (a `sync.Map`), then logs at Debug.
- **F10:** `FixVerifyAwait`'s doc states the order `Await` decides in.
- **F11, F12:** the named test rows added.

`lym gate test` (tier0) passed on `gatefixer`, `shedadapters`, `shuttleengine`, `claudeengine` and `websterengine`, and again after the self-check fixes on the three packages those touched.
The new test rows were not run against the unfixed code.

After this report was first written, the gate rejected two comment lines the round added, so no test of its own ran:
a wrapped sentence in the doc of `TestBouncer_JudgeCall_ADiedJudgeIsAnInfrastructureFailure`,
and an over-long line in `FixVerifyAwait`'s doc.
Both are rewritten, comments only, in one further commit,
and `lym gate test` (tier0) passed on `shedadapters` and `websterengine` after it.

## Self-check

- fix-consistency, MEDIUM, `waitOnly` doc still said a failing inner always marks the record terminal: fixed as proposed.
- fix-consistency, LOW, an attached judge's run error still degraded: fixed as proposed, merged with the affected-set finding.
- fix-consistency, LOW, `FixVerifyAwait` doc named a constant its package lacks: fixed with own change (qualified as `gatefixer.DeadReasonInterrupted`).
- fix-affected-set, MEDIUM, an attached judge whose wait errors still spent bounce budget: fixed as proposed (split on `attached`), with an `AttachedRunErrored` row.
- fix-proposal, LOW, a replacement spawn error vanished when the judgment passed: fixed with own change (a Warn at the spawn failure).
- fix-code-correctness, LOW, the same stale `waitOnly` sentence: fixed, merged with the first.

Noted by the forks or while judging them, outside this round's findings and left as they are:

- The bouncer's seed pass still turns a seed run error into `Stuck`.
- The entry-time judge probe still degrades when an attached judge's wait errors; it is reached only with a verdict and ledger already on disk, so that judge did give a verdict.

## Disputed

## Deferred

## Rulings
