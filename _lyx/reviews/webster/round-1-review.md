---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/fabricengine/refscanner_test.go:258
    summary: The git config row `--get --file f k` is rejected by the argument-count bound, not by the flag check it is meant to pin
  - id: F2
    severity: LOW
    class: consistency
    location: internal/websterengine/recordbatch.go:580
    summary: pathlessViolationClasses is a hand-copied list of the audit class constants with no test tying it to them, and only the fabric-reference class is exercised
  - id: F3
    severity: NIT
    class: consistency
    location: internal/fabricengine/refscanner.go:349
    summary: The `word.glob ||` operand in readOnlyLyxSegment is redundant, since holdsGlobCharacter on the dequoted text already covers it
---

The range is `git diff $(git merge-base main HEAD)..HEAD`: `status.json` gives `product.parent` as `main`, and there is no `_lyx/loom/rework` directory.
`lyx gate test` over `fabricengine`, `websterengine`, `webstercli`, `shuttleengine` and `cmd/lyx` passed.
I judged each focus item against the diff and found no defect in the first four.

- **Card 2.**
  The tokenizer rejects `$` and a backslash anywhere, an unquoted brace or parenthesis, and an unterminated quote.
  The stderr drop requires a real-space-bordered unquoted token that does not open its segment.
  A quoted `>` stays blanked, and a redirect to a real file still reaches the `>` scan.
  I found no admitted write.
- **Cards 3 and 4.**
  The `--json` help form is confined to `readOnlyLyxSegment`.
  `TestOwnJSONFlags` walks `newRoot`, so it reports the generic shed `status` verb under every mount, and the test passed with the plan's three commands.
- **Cards 1, 6, 7 and 8.**
  - Card 1: all three evidence readers skip a class-prefixed entry before link resolution.
  - Card 6: each `MerriamBase` caller adds exactly one way forward.
  - Card 7: the `begin-batch` remedy follows recover-batch's order (exhausted, retry, dead, stuck or failed) and wraps `ErrRecoveryExhausted`.
  - The CLI maps `ErrRecoveryExhausted` to `recovery_exhausted`.
- **Docs and budget.**
  `doc.go`, `refusal-spec.md` and the Master stencil are updated in the same range.
  `cmd/lyx` rises by one in `test-budget.yaml` for the single new top-level test.

### [MEDIUM] The git config flag row does not reach the flag rejection

**Location:** `internal/fabricengine/refscanner_test.go:258`
**Issue:** The row `git config --get --file f k` has four words after `config`, and `readOnlyGitSegment` requires two or three, so the count bound rejects it before the `-`-prefixed word check.
Removing that flag check would leave the row green, so it does not pin a config-touching flag as Card 5 intends.
**Fix:** Change the row to a three-word form that reaches the check, such as `git config --get --file k`, or add that form beside it.

### [LOW] The pathless class list is a hand copy with a single exercised class

**Location:** `internal/websterengine/recordbatch.go:580`
**Issue:** `pathlessViolationClasses` repeats the seven class constants of `audit.go`.
A class added later is not skipped by the three readers, and `recover-batch` breaks on its pathless entry again with `ENAMETOOLONG`.
The new rows build entries only from `ClassFabricReference`, so nothing fails if the list drifts.
**Fix:** Cover every class in an existing table, for example a `TestUncheckableReason` row per class.
Alternatively, derive the list from one place that `failOnCorrectness` and the constants share.

### [NIT] Redundant glob operand

**Location:** `internal/fabricengine/refscanner.go:349`
**Issue:** `holdsGlobCharacter(word.text)` is true for every word where `word.glob` is true, because an unquoted glob character is part of the dequoted text.
**Fix:** Drop the `word.glob ||` operand.
