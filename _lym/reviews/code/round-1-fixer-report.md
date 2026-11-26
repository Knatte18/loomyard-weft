---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed-differently
    reason: "Two proposed rows would not pin their branch. A use on card 3 after forthcoming card 2 passes without the mapping to 0, so the row's use sits on pending card 1. Quarry's alphabets reject non-canonical spellings (glyph/golang.go, glyph/csharp.go), so the parsed-ID match is pinned by a hand-built answer in TestStatusFindings."
  - id: F7
    disposition: fixed-differently
    reason: "The recording is limited to dead and in-flight batches as proposed, except that a failed or stuck batch already holding an entry still takes later edits. Without that, a second card edited after a forced card_amended failure was never listed for the next recovery (self-check, fix-proposal 5)."
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
---

# Fixer report — code round 1

## Changes

- **F1.**
  `lym webster recover-batch` takes `--answer`.
  Resuming a waiting recovery without it is refused.
  The need and answer are kept in `BatchState.ParentAnswers` and rendered into the resumed prompt.
  Master's waiting rung passes the parent's message.
- **F2.**
  A turn end with no report stays `dead`/`asking`.
  Its message is the digest's `last_message`, quoted by the recover-batch refusals and the dead-end refusal, and by Master in `stuck_reason`.
  The recovery prompt tells the strand to report a question as `status: NEEDS_PARENT`.
- **F3, F6.** New rows in `TestValidateFormat_MissingMemberFindings` and `TestStatusFindings`.
- **F4.** The last-message read runs after `StopStrand`, and its failure is a warning.
- **F5.** `TestLastTurnMessage` pins the post-offset parse and an offset past the end.
- **F7.** A rebaseline records no amendment on a failed or stuck batch that holds none.
- **F8.** `AmendedCard` and `AmendedCards` docs reworded.
- **Gate repair.** A wrapped line in the `AmendedCards` doc and an over-long line in the `Status` doc, both in `internal/runplan/state.go`, are broken into one sentence per line.
- **F9.** The plan review rubric states the unframed-run refusal.
- **F10.** `_lym/landing/summary.md` names the wider blast radius, and its change and test sections follow the fixes.
  It is under `_lym`, so it is edited and not committed.

Gates after the last change: `lym gate test` over the touched packages, `./contracts/...` and `./cmd/lym` passed at tier 0, and the tier 1 run over `webstercli`, `websterengine`, `shuttleparity` and `planglyph` passed.
After the gate repair, `lym loom lint-comments` reported no finding, and the tier 0 run over `runplan`, `websterengine` and `cmd/lym` passed.
Nothing was pushed.

## Self-check

- fix-code-correctness 1 (re-entry rule runs the `--answer` placeholder): BLOCKING, fixed.
- fix-code-correctness 2 (prefix forbids re-reporting an unmet need): MEDIUM, fixed.
- fix-code-correctness 3 (message trails the way forward, branch untested): LOW, fixed.
- fix-code-correctness 4 (transient way forwards omit `--answer`): LOW, fixed.
- fix-consistency 1 (same as fix-code-correctness 1): BLOCKING, fixed.
- fix-consistency 2 (same as fix-code-correctness 4): MEDIUM, fixed.
- fix-consistency 3 (docs define waiting as a step only): LOW, fixed-differently.
  The `need:` line in `webster-body-implementer.md` is left, since fork prompts share it;
  the recovery prefix states that such a question counts as a step.
- fix-consistency 4 (`RecoverDeps.Answer` doc): LOW, fixed.
- fix-proposal 1 (same as fix-code-correctness 1): BLOCKING, fixed.
- fix-proposal 2 (prefix contradicts the parent directive): MEDIUM, fixed.
- fix-proposal 3 (same as fix-code-correctness 4): LOW, fixed.
- fix-proposal 4 (`AmendedCard` type doc too wide): LOW, fixed.
- fix-proposal 5 (held entry not updated on failed or stuck): LOW, fixed.
- fix-affected-set 1 (same as fix-code-correctness 1): MEDIUM, fixed.
- fix-affected-set 2 (bare `recover-batch N` way forwards, `existingReportRemedy`): LOW, fixed.
- fix-affected-set 3 (same as fix-proposal 4): LOW, fixed.
- fix-affected-set 4 (re-begin drops `ParentAnswers`): LOW, fixed.

## Disputed

## Deferred

## Rulings
