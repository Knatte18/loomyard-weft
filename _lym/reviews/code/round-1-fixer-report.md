# Fixer-report, round 1

## Changes

- F1 (LOW): fixed as proposed, commit `e040a0a6`.
  The README golden row "on a board with no prime owner a beam shows its recorded owner and a plain project none" in `internal/boardengine/render_test.go` now titles P2 `Loose & <odd> | pipe`.
  Its expected summary is `P2 Loose &amp; &lt;odd&gt; | pipe`, with the pipe unescaped.
  No test function was added.
  `lym gate test ./internal/boardengine` passed: 783 passed, 0 failed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
