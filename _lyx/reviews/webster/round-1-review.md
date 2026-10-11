---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: docs/research/haiku-judge.md:48
    summary: names the retired Env.JudgeModel, JudgeEffort and JudgeVersion fields and describes a single run-wide judge key, both false after cards 1 and 4
  - id: F2
    severity: NIT
    class: design
    location: internal/shedadapters/bouncer.go:629
    summary: judgmentRefusal re-reads and re-parses round N's ledger and refuses on failure, a state recordedVerdict and harvestedVerdict already ruled out before every call
---

Range: `git diff main...HEAD` (status `product.parent` is `main`; no `_lyx/loom/rework/` directory exists, so step 4 applies).
`lyx gate test` over `shedadapters`, `shedrecipe`, `loomengine`, `loomcli`, `loomrecipe` and `shedbuild` passed.
The cards carry no Rename or Delete group, so no type-specific mechanical check applies.

Each focus question, settled against the diff:

- **Replay path (card 5).** `Call` takes `recordedVerdict` and routes through `standsAsRecorded` (`readEscalation` or `readCirclingDecision`) before `settleUnlessRefused`. A round with an escalation record or a decision file settles as recorded. `rejudge` archives the three judge outputs through `archiveStaleOutputs` and degrades with a counted, non-exempt Stuck, without calling `Bounces` or escalating. Pinned by the replay rows of `TestBouncer_JudgeCall_RefusedJudgment`.
- **Ledger keys and CONVERGED (card 5).** The dropped-key check compares keys only and names the first missing key. A previous ledger that is unreadable or unparseable logs a warning and skips the check. CONVERGED over an open BLOCKING ledger entry, a BLOCKING review finding or an unparseable review is refused. `settleUnlessRefused` serves the live harvest, the entry-time attach and the replay, so the refusal holds on all paths.
- **Row-keyed models (cards 3, 4).** `bouncerEntry` fails on a row with no `Env.RowBouncerModels` entry and rejects `model`, `effort` and `version` as unknown keys. `loomcli` fills the map for the three Bouncer rows. No Go callers of the old `JudgeModel` fields remain. The one stale prose reference is F1.
- **Empty-key fallback (cards 1, 2, 4).** `resolveSegmentBouncerModels` and `ResolveRework` implement the documented fallback chain, `rework` falling back to `plan`. `LoadConfig` validates the eight new keys only when non-empty. `wire` also resolves rework once at run start.
- **Usage record (cards 6, 7).** `Review` and `Fix` are `*halfUsage` with `omitempty`, so a Burler-written record keeps its on-disk form. `readCompletedRoundUsage` serves both readers, `computeLensFacts` and `lensesRunUnderFan`. `mergeRoundUsage` is the single write path, and the Burler's `doneExit` overwrites only its own fields, so earlier seed and judge entries survive. The Bouncer records the seed pass, the re-bounce seed attach, the judge pass (including a refused one), the judge's own attach and the entry-time attach. A Run error records nothing.
- **Test economy.** `TestBouncer_JudgeCall_RefusedJudgment` and `TestBouncer_UsageRecord` are table-driven and each covers behavior no existing test covers. The `RowBouncerModels` rows in `shedrecipe` replace the `JudgeModel` rows rather than duplicating them. I found no redundant test.

### [LOW] Research note still describes the retired run-wide judge fields

**Location:** `docs/research/haiku-judge.md:48`
**Issue:** Section "How the judge's model is set" says one `judge` key covers every row and that `shedrecipe.Env.JudgeModel`/`JudgeEffort`/`JudgeVersion` carry it. Card 4 removes those fields, and card 1 adds the per-segment and seed keys. The note also says `bouncerEntry` accepts per-row `model`, `effort` and `version`, which it now refuses. Card 4 requires the retired keys to leave the docs as well as the code.
**Fix:** Reword the section to the `RowBouncerModels` threading and the new keys, or mark the note as a dated snapshot that predates this change.

### [NIT] Unreachable ledger-read refusal in judgmentRefusal

**Location:** `internal/shedadapters/bouncer.go:629`
**Issue:** Every call site reaches `judgmentRefusal` after `harvestedVerdict` or `recordedVerdict` has returned true. Both require round N's ledger to read and parse, so the opening `os.ReadFile` and `parseLedger` error returns ("the ledger is unreadable", "the ledger does not parse") cannot fire. This is a branch for a state the code's own guarantees rule out. It is harmless but untested.
**Fix:** Either keep the parsed ledger from the earlier read, or document why the re-read stays. Do not add a test for the dead branch.
